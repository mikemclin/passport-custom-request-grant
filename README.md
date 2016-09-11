## Install

Install with composer...  `composer require mikemclin/passport-custom-request-grant`

## Setup

* Add `MikeMcLin\Passport\CustomRequestGrantProvider` to your list of providers **after** `Laravel\Passport\PassportServiceProvider`.
* Add `getUserEntityByRequest($request)` method to your `User` model (or whatever model you have configured to work with Passport).
    * The method should accept an `Illuminate\Http\Request` object.
    * You should authorize and retrieve user based on this request
    * If you find that the request met your requirement, return the User model.
    * If the request did not satisfy your requirement, return `null`

## How to use

* Make a **POST** request to `https://your-site.com/oauth/token`, just like you would a **Password** or **Refresh** grant.
* The POST body should contain `grant_type` = `custom_request`.
* The request will get routed to your `User::getUserEntityByRequest()` function, where you will determine if access should be granted or not.
* An `access_token` and `refresh_token` will be returned if successful.

### Example

Here is what a `User::byPassportCustomRequest()` method might look like...

```php
/**
 * Verify and retrieve user by custom token request.
 *
 * @param \Illuminate\Http\Request $request
 *
 * @return \Illuminate\Database\Eloquent\Model|null
 * @throws \League\OAuth2\Server\Exception\OAuthServerException
 */
public function byPassportCustomRequest(Request $request)
{
    try {
        if ($request->get('sso_token')) {
            return $this->bySsoToken($request->get('sso_token'));
        }
    } catch (\Exception $e) {
        throw OAuthServerException::serverError($e->getMessage());
    }
    return null;
}
```

In this example, the app is able to authenticate a user based on an `sso_token` property from a submitted JSON payload.  The `bySsoToken` is this app's way of doing that.  It will return `null` or a user object.  It also might throw exceptions explaining why the token is invalid.  The `byPassportCustomRequest` catches any of those exceptions and converts them to appropriate OAuth exception type.  If an `ssoToken` is not present on the request payload, then we return `null` which returns an **invalid_credentials** error response:

```json
{
  "error": "invalid_credentials",
  "message": "The user credentials were incorrect."
}
```
