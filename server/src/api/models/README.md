```JS
//user.server.model.js

'use strict';

var mongoose = require('mongoose'),
  bcrypt   = require('bcrypt-nodejs'),
  Schema   = mongoose.Schema,
  Promise  = require('bluebird');



var validateLocalStrategyProperty = function(property) {
  return (property.length);
};

var validateLocalStrategyPassword = function(password) {
  return (this.provider !== 'local' || (password && password.length > 1));
};


var UserSchema = new Schema({
    firstName: {
      type: String,
      trim: true,
      default: '',
      validate: [validateLocalStrategyProperty, 'Please fill in your first name']
    },
    lastName: {
      type: String,
      trim: true,
      default: '',
      validate: [validateLocalStrategyProperty, 'Please fill in your last name']
    },
    displayName: {
      type: String,
      trim: true
    },
    email: {
      type: String,
      trim: true,
      unique: 'Email already exists',
      validate: [validateLocalStrategyProperty, 'Please fill in your email'],
      match: [/.+\@.+\..+/, 'Please enter a valid email address']
    },
    username: {
      type: String,
      unique: 'Username already exists',
      required: 'Please fill in a username',
      trim: true
    },
    password: {
      type: String,
      default: '',
      validate: [validateLocalStrategyPassword, 'Password should not be empty']
    },
    status: {
      type: String,
      default: "active"
    },
    profileImageURL: {
      type: String,
      default: '/img/profile/default.png'
    },
    designation: {
      type: String
    },
    webUrl: {
      type: String
    },
    stackOverflowProfile: {
      type: String
    },
    gitHubProfile: {
      type: String
    },
    linkedInProfile: {
      type: String
    },
    provider: {
      type: String,
      required: 'Provider is required',
      default: 'local'
    },
    providerData: {},
    additionalProvidersData: {},
    roles: {
      type: [{
        type: String,
        enum: ['user', 'admin']
      }],
      default: ['user'],
      required: 'Please provide role'
    },
    /* For reset password */
    resetPasswordToken: {
      type: String
    },
    resetPasswordExpires: {
      type: Date
    },
    sessions: {
      sessionToken : String,
      createDateTime:{
        type: Date,
        default: Date.now
      },
      lastAccessDateTime:{
        type: Date,
        default: Date.now
      }
    }
  },
  {timestamps: {
    createdAt: 'createdDate',
    updatedAt: 'updatedDate'
  }});

UserSchema.methods.generateHash = function(password) {
  return bcrypt.hashSync(password, bcrypt.genSaltSync(8), null);
};

UserSchema.methods.verifiedPassword = function(password) {
  return bcrypt.compareSync(password, this.password);
};

UserSchema.pre('save', function(next) {
  if (this.password && this.password.length > 1) {
    this.password = this.generateHash(this.password);
  }
  next();
});

UserSchema.statics.findUniqueUsername = function (username, suffix, callback) {
  var _this = this;
  return new Promise((resolve) => {
    Promise.coroutine(function*() {
      try {
        var possibleUsername = username.toLowerCase() + (suffix || '');
        _this.findOne({
          username: possibleUsername
        }, function (err, user) {
          if (!err) {
            if (!user) {
              return resolve(possibleUsername);
            } else {
              return _this.findUniqueUsername(username, (suffix || 0) + 1, callback);
            }
          } else {
            return resolve(null);
          }
        });
      } catch (err) {
        return resolve(null);
      }
    })();
  });
};

module.exports = mongoose.model('User', UserSchema);
```
