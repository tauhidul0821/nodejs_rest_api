```JS
'use strict';

var utility     = require('../helpers/utility'),
  constants     = require('../helpers/constants'),
  Promise       = require('bluebird'),
  UserService   = require('../services/user.server.service');

exports.signUp = function(req, res) {
  Promise.coroutine(function*() {
    var response = yield UserService.signUp(req);
    if(response.success) {
      utility.logMessage('info',
        {
          id: constants.logging.actions.userSignUp,
          action: constants.logging.actions.userSignUp,
          location: constants.logging.locations.userServerController,
          req: req ,
          status: constants.logging.status.success
        },{data: response});
    } else {
      utility.logMessage('error',
        {
          id: constants.logging.actions.userSignUp,
          action: constants.logging.actions.userSignUp,
          location: constants.logging.locations.userServerController,
          req: req,
          status: constants.logging.status.failed
        }, response.errorMsg);
    }
    return res.status(200).json(response);
  })();
};

exports.signIn = function(req, res) {
  Promise.coroutine(function*() {
    var response = yield UserService.signIn(req, res);
    if(response.success) {
      utility.logMessage('info',
        {
          id: constants.logging.actions.userSignIn,
          action: constants.logging.actions.userSignIn,
          location: constants.logging.locations.userServerController,
          req: req ,
          status: constants.logging.status.success
        },{data: response});
    } else {
      utility.logMessage('error',
        {
          id: constants.logging.actions.userSignIn,
          action: constants.logging.actions.userSignIn,
          location: constants.logging.locations.userServerController,
          req: req,
          status: constants.logging.status.failed
        }, response.errorMsg);
    }
    return res.status(200).json(response);
  })();
};

exports.signOut = function(req, res) {
  Promise.coroutine(function*() {
    var response = yield UserService.signOut(req);
    if(response.success) {
      utility.logMessage('info',
        {
          id: constants.logging.actions.signOut,
          action: constants.logging.actions.signOut,
          location: constants.logging.locations.userServerController,
          req: req ,
          status: constants.logging.status.success
        },{data: response});
    } else {
      utility.logMessage('error',
        {
          id: constants.logging.actions.signOut,
          action: constants.logging.actions.signOut,
          location: constants.logging.locations.userServerController,
          req: req,
          status: constants.logging.status.failed
        }, response.errorMsg);
    }
    return res.status(200).json(response);
  })();
};

/**
 * OAuth provider call
 */
exports.oauthCall = function (strategy, scope) {
  return UserService.oauthCall(strategy, scope);
};

exports.oauthCallback = function (strategy) {
  return UserService.oauthCallback(strategy);
};

exports.saveOAuthUserProfile = function(req, providerUserProfile, done) {
  Promise.coroutine(function*() {
    var response = yield UserService.saveOAuthUserProfile(req, providerUserProfile);
    if(response.success) {
      utility.logMessage('info',
        {
          id: constants.logging.actions.saveOAuthUserProfile,
          action: constants.logging.actions.saveOAuthUserProfile,
          location: constants.logging.locations.userServerController,
          req: req ,
          status: constants.logging.status.success
        },{data: response});
    } else {
      utility.logMessage('error',
        {
          id: constants.logging.actions.saveOAuthUserProfile,
          action: constants.logging.actions.saveOAuthUserProfile,
          location: constants.logging.locations.userServerController,
          req: req,
          status: constants.logging.status.failed
        }, response.errorMsg);
    }
    return done(response.err, response.user, response.info);
  })();
};

exports.getUserProfile = function(req, res) {
  Promise.coroutine(function*() {
    var response = yield UserService.getUserProfile(req);
    if(response.success) {
      utility.logMessage('info',
        {
          id: constants.logging.actions.getUserProfile,
          action: constants.logging.actions.getUserProfile,
          location: constants.logging.locations.userServerController,
          req: req ,
          status: constants.logging.status.success
        },{user: response.user._id});
    } else {
      utility.logMessage('error',
        {
          id: constants.logging.actions.getUserProfile,
          action: constants.logging.actions.getUserProfile,
          location: constants.logging.locations.userServerController,
          req: req,
          status: constants.logging.status.failed
        }, response.errorMsg);
    }
    return res.status(200).json(response);
  })();
};

exports.updateUserProfile = function(req, res) {
  Promise.coroutine(function*() {
    var response = yield UserService.updateUserProfile(req);
    if(response.success) {
      utility.logMessage('info',
        {
          id: constants.logging.actions.updateUserProfile,
          action: constants.logging.actions.updateUserProfile,
          location: constants.logging.locations.userServerController,
          req: req ,
          status: constants.logging.status.success
        },{user: response.user._id});
    } else {
      utility.logMessage('error',
        {
          id: constants.logging.actions.updateUserProfile,
          action: constants.logging.actions.updateUserProfile,
          location: constants.logging.locations.userServerController,
          req: req,
          status: constants.logging.status.failed
        }, response.errorMsg);
    }
    return res.status(200).json(response);
  })();
};

exports.list = function(req, res) {
  Promise.coroutine(function*() {
    var response = yield UserService.list();
    if(response.success) {
      utility.logMessage('info',
        {
          id: constants.logging.actions.userList,
          action: constants.logging.actions.userList,
          location: constants.logging.locations.userServerController,
          req: req ,
          status: constants.logging.status.success
        },{noOfUserReturned: response.users.length});
    } else {
      utility.logMessage('error',
        {
          id: constants.logging.actions.userList,
          action: constants.logging.actions.userList,
          location: constants.logging.locations.userServerController,
          req: req,
          status: constants.logging.status.failed
        }, response.errorMsg);
    }
    return res.status(200).json(response);
  })();
};

exports.getUserById = function(req, res) {
  Promise.coroutine(function*() {
    var response = yield UserService.getUserById(req.params);
    if(response.success) {
      utility.logMessage('info',
        {
          id: constants.logging.actions.getUser,
          action: constants.logging.actions.getUser,
          location: constants.logging.locations.userServerController,
          req: req ,
          status: constants.logging.status.success
        },{data: response});
    } else {
      utility.logMessage('error',
        {
          id: constants.logging.actions.getUser,
          action: constants.logging.actions.getUser,
          location: constants.logging.locations.userServerController,
          req: req,
          status: constants.logging.status.failed
        }, response.errorMsg);
    }
    return res.status(200).json(response);
  })();
};
```
