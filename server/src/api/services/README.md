```JS
// user.server.service.js
'use strict';

var User          = require('../models/user.server.model'),
  errorResolver   = require('../helpers/errorResolver'),
  Promise         = require('bluebird'),
  passport        = require('passport'),
  _               = require('lodash');

// URLs for which user can't be redirected on signin
var noReturnUrls = [
  '/login',
  '/registration'
];

exports.signUp = function(req) {
  return new Promise((resolve) => {
    Promise.coroutine(function*() {
      try {
        let reqBody = req.body;
        var newUser = new User(reqBody);
        newUser.email = reqBody.email.toLowerCase();
        newUser.displayName = reqBody.firstName +' '+ reqBody.lastName;
        newUser.username = newUser.email;

        var UVError = newUser.validateSync();
        if(UVError) {
          return resolve({ success: false, errorMsg: errorResolver.resolve(UVError) });
        }

        yield newUser.save();

        newUser.password = undefined;

        req.login(newUser, function(err) {
          if (err) {
            return resolve({success: false, errorMsg: errorResolver.resolve(err)});
          } else {
            return resolve({success: true, user: newUser});
          }
        });
      } catch (err) {
        return resolve({ success: false, errorMsg: errorResolver.resolve(err) });
      }
    })();
  });
};

exports.signIn = function(req, res) {
  return new Promise((resolve) => {
    Promise.coroutine(function*() {
      try {
        passport.authenticate('local', function(err, user, info) {
          if (err) {
            return resolve({success: false, errorMsg: errorResolver.resolve(err)});
          } else if(!user){
            return resolve({success: false, errorMsg: info.message});
          }else {
            // Remove sensitive data before login
            user.password = undefined;

            req.login(user, function(err) {
              if (err) {
                return resolve({success: false, errorMsg: errorResolver.resolve(err)});
              } else {
                return resolve({success: true, user: user});
              }
            });
          }
        })(req, res);
      } catch (err) {
        return resolve({ success: false, errorMsg: errorResolver.resolve(err) });
      }
    })();
  });
};

exports.signOut = function(req) {
  return new Promise((resolve) => {
    Promise.coroutine(function*() {
      try {
        req.logout();
        return resolve({ success: true});
      } catch (err) {
        return resolve({ success: false, errorMsg: errorResolver.resolve(err) });
      }
    })();
  });
};

/**
 * OAuth provider call
 */
exports.oauthCall = function (strategy, scope) {
  return function (req, res, next) {
    if (req.query && req.query.redirect_to)
      req.session.redirect_to = req.query.redirect_to;

    // Authenticate
    passport.authenticate(strategy, scope)(req, res, next);
  };
};

/**
 * OAuth callback
 */
exports.oauthCallback = function (strategy) {
  return function (req, res, next) {

    // info.redirect_to contains inteded redirect path
    passport.authenticate(strategy, function (err, user, info) {
      if (err) {
        console.log('auth error== ',err);
        return res.redirect('/login?err=' + encodeURIComponent(err));
      }
      if (!user) {
        return res.redirect('/login');
      }
      req.login(user, function (err) {
        if (err) {
          return res.redirect('/login');
        }

        //return res.status(200).send({user: user});
        return res.redirect(info.redirect_to || '/?userId='+user._id);
      });
    })(req, res, next);
  };
};


/**
 * Helper function to save or update a OAuth user profile
 */
exports.saveOAuthUserProfile = function (req, providerUserProfile) {
  return new Promise((resolve) => {
    Promise.coroutine(function*() {
      // Setup info object
      let info = {};
      try {
        // Set redirection path on session.
        // Do not redirect to a signin or signup page
        if (noReturnUrls.indexOf(req.session.redirect_to) === -1)
          info.redirect_to = req.session.redirect_to;

        if (!req.user) {
          // Define a search query fields
          var searchMainProviderIdentifierField = 'providerData.' + providerUserProfile.providerIdentifierField;
          var searchAdditionalProviderIdentifierField = 'additionalProvidersData.' + providerUserProfile.provider + '.' + providerUserProfile.providerIdentifierField;

          // Define main provider search query
          var mainProviderSearchQuery = {};
          mainProviderSearchQuery.provider = providerUserProfile.provider;
          mainProviderSearchQuery[searchMainProviderIdentifierField] = providerUserProfile.providerData[providerUserProfile.providerIdentifierField];

          // Define additional provider search query
          var additionalProviderSearchQuery = {};
          additionalProviderSearchQuery[searchAdditionalProviderIdentifierField] = providerUserProfile.providerData[providerUserProfile.providerIdentifierField];

          // Define a search query to find existing user with current provider profile
          var searchQuery = {
            $or: [mainProviderSearchQuery, additionalProviderSearchQuery]
          };

          let user = yield User.findOne(searchQuery);
          if (!user) {
            var possibleUsername = providerUserProfile.username || ((providerUserProfile.email) ? providerUserProfile.email.split('@')[0] : '');

            let availableUsername = yield User.findUniqueUsername(possibleUsername, null);
            user = new User(providerUserProfile);
            // Email intentionally added later to allow defaults (sparse settings) to be applid.
            // Handles case where no email is supplied.
            // See comment: https://github.com/meanjs/mean/pull/1495#issuecomment-246090193
            user.email = providerUserProfile.email;
            user.username = availableUsername;
            yield user.save();
            return resolve({err: null, user: user, info: info});
          } else {
            return resolve({err: null, user: user, info: info});
          }
        } else {
          // User is already logged in, join the provider data to the existing user
          let user = req.user;

          // Check if user exists, is not signed in using this provider, and doesn't have that provider data already configured
          if (user.provider !== providerUserProfile.provider && (!user.additionalProvidersData || !user.additionalProvidersData[providerUserProfile.provider])) {
            // Add the provider data to the additional provider data field
            if (!user.additionalProvidersData) {
              user.additionalProvidersData = {};
            }

            user.additionalProvidersData[providerUserProfile.provider] = providerUserProfile.providerData;

            // Then tell mongoose that we've updated the additionalProvidersData field
            user.markModified('additionalProvidersData');

            // And save the user
            yield user.save();
            return resolve({err: null, user: user, info: info});
          } else {
            return resolve({err: 'User is already connected using this provider', user: user});
          }
        }
      } catch (err) {
        return resolve({ err: err, user: req.user, info: info });
      }
    })();
  });
};

exports.getUserProfile = function(req) {
  return new Promise((resolve) => {
    Promise.coroutine(function*() {
      try {
        let userId = req.params.userId;
        if(req.user && req.user._id.toString() === userId) {
          let user = yield User.findOne({_id: userId}).select({provider: 0, providerData: 0, password: 0}).exec();
          return resolve({success: true, user: user});
        } else {
          return resolve({success: false, errorMsg: 'Invalid user'});
        }
      } catch (err) {
        return resolve({ success: false, errorMsg: errorResolver.resolve(err) });
      }
    })();
  });
};

exports.updateUserProfile = function(req) {
  return new Promise((resolve) => {
    Promise.coroutine(function*() {
      try {
        let user = req.user;
        let userId = req.params.userId;
        if(!user || (user && user._id.toString() !== userId)) {
          return resolve({ success: false, errorMsg: "Invalid user ID" });
        }
        var whiteListedFields = ['firstName', 'lastName', 'email', 'username', 'designation', 'webUrl', 'stackOverflowProfile', 'gitHubProfile', 'linkedInProfile'];
        user = _.extend(user, _.pick(req.body, whiteListedFields));
        user.displayName = user.firstName + ' ' + user.lastName;
        yield user.save();
        req.login(user, function (err) {
          if (err) {
            return resolve({ success: false, errorMsg: errorResolver.resolve(err) });
          } else {
            return resolve({ success: true, user: user });
          }
        });
      } catch (err) {
        return resolve({ success: false, errorMsg: errorResolver.resolve(err) });
      }
    })();
  });
};

exports.list = function() {
  return new Promise((resolve) => {
    Promise.coroutine(function*() {
      try {
        let users = yield User.find({}).exec();
        return resolve({success: true, users: users});
      } catch (err) {
        return resolve({ success: false, errorMsg: errorResolver.resolve(err) });
      }
    })();
  });
};

exports.getUserById = function(reqParams) {
  return new Promise((resolve) => {
    Promise.coroutine(function*() {
      try {
        let query = {};
        if(reqParams.id) {
          query._id =  reqParams.id;
        }
        let user = yield User.findOne(query, {password: -1, providerData: -1, session: -1});
        var successData = {
          success: true,
          user: user
        };
        return resolve(successData);
      } catch (err) {
        return resolve({ success: false, errorMsg: errorResolver.resolve(err) });
      }
    })();
  });
};
```
