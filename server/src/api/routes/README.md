```JS
// user.server.routes.js

'use strict';

var userController = require('../controllers/user.server.controller'),
  config           = require('../config/config'),
  auth             = require('../helpers/authorization');

module.exports = function(app) {
  var url = config.app.baseUrl;

  app.route(url + 'registration/')
    .post(userController.signUp);

  app.route(url + 'login/')
    .post(userController.signIn);

  app.route(url + 'logout/')
    .post(userController.signOut);

  // Setting the github oauth routes
  app.route(url + 'auth/github')
    .get(userController.oauthCall('github'));

  app.route(url + 'auth/github/callback')
    .get(userController.oauthCallback('github'));

  app.route(url + 'users/')
    .get(userController.list);

  app.route(url + 'users/profile/:userId')
    .get(userController.getUserProfile)
    .put(userController.updateUserProfile);

  app.route(url + 'users/:id')
    .get(userController.getUserById);

};
```
