---
layout: post
title: Making authentication logic for Flutter + Nestjs
date:  2020-10-21 
description: 
tags:
- web
- flutter
- dart
- nestjs
- nodejs
- typescript
permalink: authentication-with-flutter-nestjs

---

Few days ago, I've planned to design some side project, which is based on mobile application. This is the record about how I've implement authentication logic through application side to back-end side.

I've once wrote about authentication with golang, but in this time I've decided to use nodejs. Still yet golang was not handy to me, and because I've changed my position on work, it requires much time to get used on new work(and studying new programming language).

Moreover, I've decided use Flutter, which I've never used before, so couldn't have much time to assign a time for get used to golang.


## Why Flutter?
If the performance is not a high priority for application, it's inconvenient to develop apps for iOS and Android separately. Luckly there are several ways to develop application for iOS/Android in same code base.

![image](/assets/post_img/authentication-with-flutter-nestjs/flutter-main-page.png)

There are other frameworks(Xamarin, React-Native) to make this done, but lots of performance benchmark(it's pretty close to native!), growth of community, and support from Google make me to choose Flutter. Also, there were similarities with 'component' based development at front-end.

One sad thing(maybe good thing) is there are no other choice than using `Dart` for development, but thought it worths to go on with this.


## Why Nestjs

![image](/assets/post_img/authentication-with-flutter-nestjs/nest-main-page.png)

NestJS is one of web framework based on NodeJS, like Express, Koa. It has rapidly grow-up in few years and become one of most trendy project, by offering user to build modulized, looseley coupled architecture with less effort.

Check the small piece of code:
```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

As you can see, it actively supports `Decorator` for clean code. Maybe you can remind of `Spring` framework in Java, or `Angular` framework. It also supports TypeScript by default, for type-safety application.

I was using Express for years but had chance to use this for few month, and impressed in development efficiency. Now I'm not with NodeJS, but wanted to get more deeply with this framework, by apply in this personal project.


## Authentication logic for Flutter
These are the logic for basic authentication.

1. Open application
2. Check state of authentication with stored JWT
   1. If JWT is available, show main page
   2. If not, move to login page
3. In 2-2, login with ID/PW, and store JWT to local storage


You can just call API for each process and check state separately, but let's try to use 'provider pattern' to keep state in unified logic.


### Define API
First let's define APIs to get/post login informations from server.

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

// flutter
import 'package:http/http.dart' as http;
import 'package:logging/logging.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

// local
import 'package:app/services/api_response.dart';
import 'package:app/services/api_error.dart';
import 'package:app/models/user.dart';

String _baseUrl = "base-url-to-call-api";

Future<ApiResponse> authenticateUser(String username, String password) async {
  ApiResponse _apiResponse = new ApiResponse();
  final _storage = new FlutterSecureStorage();

  try {
    final resp = await http.post('${_baseUrl}api/auth/login', body: {
      'username': username,
      'password': password
    });
    
    if (resp.statusCode == 200 || resp.statusCode == 201) {
      Map mapped= json.decode(resp.body);
      String token = mapped["access_token"];
      await _storage.write(key: "accessToken", value: token);

      String tok = await _storage.read(key: "accessToken");
      return await getProfile();
      
    } else {
      _apiResponse.ApiError = ApiError.fromJson(json.decode(resp.body));
    }
  } catch (e) {
    _apiResponse.ApiError = ApiError(error: "Unknown server error. Please retry");
  }

  return _apiResponse;
}

Future<ApiResponse> getProfile() async {
  final _storage = new FlutterSecureStorage();
  final token = await _storage.read(key: 'accessToken');
  ApiResponse _apiResponse = new ApiResponse();

  try {
    final profile = await http.get(
      '${_baseUrl}api/users/profile', 
      headers: {HttpHeaders.authorizationHeader: 'Bearer ${token}'}
    );

    if (profile.statusCode == 200 || profile.statusCode == 201) {
      _apiResponse.Data = Users.fromJson(json.decode(profile.body));
    } else {
      _apiResponse.ApiError = ApiError.fromJson(json.decode(profile.body));
    }
  } on Exception {
    _apiResponse.ApiError = ApiError(error: "Unknown server error. Please retry");
  }

  return _apiResponse;
}
```

I've used wrapper class for response handler(ApiResponse, ApiError). About this, you can find more in [here](https://mundanecode.com/posts/flutter-restapi-login/). It is one kind of method to keep common rule for response handling.  This post will only focus on managing login state.

There are 2 APIs here:
> - ${_baseUrl}api/auth/login -> login with ID/PW
> - ${_baseUrl}api/users/profile -> get profile with token stored in device. It is to check user is logged in or not.


### with provider pattern
Provider pattern is one of design pattern, to manage application state. In this pattern, states are being managed in separate module, and views which requires these information will check states from this module. If you're familiar with `react-redux` or `vuex`, you can understand more quickly.

For implementation, you need to add `provider` module.


```yaml
dependencies:
  ...
  provider: ^4.3.2
```

First, let's define 'state' of login. Basically, you can think of 'loading', 'success', 'fail',　but you can divide into more segments if needed.
```dart
enum LoginState { Loading, Success, Fail }
```

Now will make provider class. For this, you need to extend `ChangeNotifier`. This is a simple class included in the Flutter SDK which provides change notification to its listeners. By extending this, user can subscribe to its change.

```dart
class AuthProviders with ChangeNotifier {
  
  LoginState _state = LoginState.Fail;
  LoginState get state => _state;

  AuthProviders() {
    this._state = LoginState.Loading;

    getProfile().then((resp) {
      ApiResponse response = resp;
      if (response != null && response.Data != null) {
        _log.info('success auth api call');
        this._state = LoginState.Success;
      } else {
        _log.info('fail auth api call');
        this._state = LoginState.Fail;
      }

      notifyListeners();

    }).catchError((onError) {
      this._state = LoginState.Fail;
      notifyListeners();
    });
  }

  Future<bool> login(String id, String password) async {
    this._state = LoginState.Loading;
    notifyListeners();

    try {
        
      await authenticateUser(id, password);
    } catch (e) {
      this._state = LoginState.Fail;
      
    }

    notifyListeners();
  }
}
```

First, you can see line `LoginState _state = LoginState.Fail`, for initiating state. Init value is 'fail' because it didn't started(or you can make state 'Init' for this).

When user try to call API to login or check state, it will change state as `Loading` until it gets the result(Success/Fail).


## JWT inside Nestjs
Nestjs offers various features for web server development as module, and so as [authentication](https://docs.nestjs.com/security/authentication). I'm able to check whether user is authenticated or not by checking value inside 'Bearer Token' is valid one. It can be checked by parsing header with code, but Nestjs offers smarter way with `passport.js` module.

In Nestjs, it offers `Guards` class annotation. As you can expect by the name, it makes whether a given request will be handled by the route handler or not, depending on certain conditions (like permissions, roles, ACLs, etc.) present at run-time. This is pretty useful function for authentication logic.

Also, it makes user to integrate `Strategy` which is offered from `passport` module.

> Passport has a rich ecosystem of strategies that implement various authentication mechanisms. While simple in concept, the set of Passport strategies you can choose from is large and presents a lot of variety

First setup scaffold with [Nest CLI](https://docs.nestjs.com/first-steps), install additional modules for this:

```
$ npm install --save passport passport-jwt @nestjs/passport @nestjs/jwt
$ npm install --save-dev @types/passport-jwt
```

`@nestjs/passport` is extention for Nestjs, which will offer extendable module for authentication.

Before making authentication guards, make API controllers for API defined in flutter app.

```typescript
import {
  Controller,
  Get,
  Post,
  Logger,
  Request,
  UseGuards,
} from '@nestjs/common'
import { AuthService } from './auth/auth.service'

@Controller()
export class AppController {
  constructor(private readonly authService: AuthService) {}

  // POST: login user
  @Post('api/auth/login')
  async postLogin(@Request() req) {
    return this.authService.login(req.user)
  }

  // GET : return user profile
  @Get('api/user/profile')
  async profile(@Request() req) {
    return this.authService.getProfile(req.user)
  }
}
```

I'll make `api/user/profile` to be blocked if token is not included or invalid, and `api/auth/login` to return token for this.

First, for login:

```typescript
import { Injectable, Logger } from '@nestjs/common'
import { JwtService } from '@nestjs/jwt'

@Injectable()
export class AuthService {
  private readonly logger = new Logger(AuthService.name)
  constructor(
    private readonly jwtService: JwtService,
  ) {}

  async login(user: any) {
    const payload = { username: user.username, sub: user.password }
    return {
      access_token: this.jwtService.sign(payload),
    }
  }
}
```

If `login` has been succeed, it will return `access_token` which will be used for authentication.
Now make `Guards` for this...

[jwt-auth.guard.ts]
```typescript
import { Injectable } from '@nestjs/common'
import { AuthGuard } from '@nestjs/passport'

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}
```

and define `Strategy` for JWT:

[jwt-strategy.ts]
```typescript
import { ExtractJwt, Strategy } from 'passport-jwt'
import { PassportStrategy } from '@nestjs/passport'
import { Injectable } from '@nestjs/common'

const jwtConstants = {
  secret: 'secretKey',
}

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: jwtConstants.secret,
    })
  }

  async validate(payload: any) {
    console.log(payload)
    return { userId: payload.sub, username: payload.username }
  }
}
```

and call with `UseGuards`

```typescript
import {
  Controller,
  Get,
  Request,
  UseGuards,
} from '@nestjs/common'
import { JwtAuthGuard } from '../jwt-auth.guard'
// ...
  @UseGuards(JwtAuthGuard)
  @Get('api/user/profile')
  async profile(@Request() req) {
    return this.authService.getProfile(req.user)
  }
```

This will make `api/user/profile` protected from unauthorized call...

```
$ curl http://localhost:3000/api/user/profile
$ # result -> {"statusCode":401,"error":"Unauthorized"}

$ curl http://localhost:3000/api/user/profile -H "Authorization: Bearer <received-auth-token>"
$ # result -> {"userId": "user-id", "username": "user-name"}
```

You can find full example [here](https://docs.nestjs.com/security/authentication).



## Reference
* <https://flutter.dev/docs>
* <https://mundanecode.com/posts/flutter-restapi-login/>
* <https://docs.nestjs.com/>
* <https://docs.nestjs.com/security/authentication>

