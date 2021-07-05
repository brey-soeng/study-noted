# Swagger documentation for Laravel API

API documentation becomes very necessary when you split the team into Backend and Frontend. And even more when you divide your monorepo into parts or even microservices.

This package is a wrapper of [Swagger-php](https://github.com/zircote/swagger-php) and [swagger-ui](https://github.com/swagger-api/swagger-ui) adapted to work with Laravel.

```
composer require "darkaonline/l5-swagger"
```

Then publish config and all view files:

```
php artisan vendor:publish --provider "L5Swagger\L5SwaggerServiceProvider"
```

That means that you have to create that notation first. So let’s add it. I prefer creating Abstract controller for an API, but you can add this to app/Http/Controllers/Controller.php

It will return error if you doesn't add  **@OA\Info(...)** below: 

```
/**
 * @OA\Info(
 *    title="Your super  ApplicationAPI", 
 *    version="1.0.0",
 * )
 */
class Controller extends BaseController
{
    use AuthorizesRequests, DispatchesJobs, ValidatesRequests;
}
```

Next, we need to add docs for at least one route. Let’s add it for app/Http/Controllers/Auth/LoginController.php:

```
/**
 * @OA\Post(
 * path="/api/auth/login",
 * summary="Sign in",
 * description="Login by email, password",
 * operationId="authLogin",
 * tags={"auth"},
 * @OA\RequestBody(
 *    required=true,
 *    description="Pass user credentials",
 *    @OA\JsonContent(
 *       required={"username","password"},
 *       @OA\Property(property="username", type="string", format="email", example="admin"),
 *       @OA\Property(property="password", type="string", format="password", example="123456"),
 *       @OA\Property(property="persistent", type="boolean", example="true"),
 *    ),
 * ),
 *
 * @OA\Response(
 *    response=422,
 *    description="Wrong credentials response",
 *    @OA\JsonContent(
 *       @OA\Property(property="message", type="string", example="Sorry, wrong email address or password. Please try again")
 *        )
 *     )
 * )
 */
```

Those are the most important to start. Now if you try to create docs using this command : every update you code you need to command code below:

```
php artisan l5-swagger:generate
```

![](https://i.loli.net/2021/07/02/ESQDbTOVaMWACH5.png)

Another tip for authorizing some URLs with JWT. You can add a security parameter this way:

```
* security={ {"bearer": {} }},
```

First you need to add code to any middleware to add jwt token to authorized

```
/**
 * @OA\SecurityScheme(
 *     type="http",
 *     description="Login with email and password to get the authentication token",
 *     name="Token based Based",
 *     in="header",
 *     scheme="bearer",
 *     bearerFormat="JWT",
 *     securityScheme="bearer",
 * )
 */
```

![](https://i.loli.net/2021/07/02/82EN9jrfeFI3JkL.png)

then you can use block code below:

```
/**
* @OA\Post (
 *     path="/api/auth/logout",
 *     summary="Logout user information",
 *     description="Logout user useing jwt token",
 *     operationId="authLogout",
 *     tags={"auth"},
 *     security={{ "bearer": {} }},
 * @OA\Response(
 *     response=200,
 *     description="Success"
 *  ),
 *  @OA\Response(
 *     response=401,
 *     description="User should be authorized to get profile information",
 *      @OA\JsonContent(
 *           @OA\Property(property="message", type="string", example="Not authorized"),
 *      )
 *    )
 * ),
 */
```

https://ikolodiy.com/posts/laravel-swagger-tips-examples

https://swagger.io/docs/specification/about/

![](https://i.loli.net/2021/07/02/7TnhGZpo1vLEywV.png)

