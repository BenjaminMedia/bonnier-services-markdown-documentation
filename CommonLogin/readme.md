# CommonLogin Integration Documentation
If you want a better general understanding of Oauth2, you can read more here: https://oauth.net/2/

## Authorization Flow
Once a client has been created and given to you by Bonnier, you can use the client ID and secret to request authorization codes and access tokens from your application.

A typical flow goes through three steps.

1. Redirecting to the login service for authorization
2. Converting Authorization Code to Access Token
3. Refreshing Access Tokens

### 1. Redirecting For Authorization
Once the user initiates a login flow by clicking a login button in your application, then you should make a redirect request to `/oauth/authorize` route like so:

```php
Route::get('/redirect', function () {
    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://example.com/callback',
        'response_type' => 'code',
        'state' => 'your-state-data'
        'scope' => '',
    ]);
	
    return redirect('http://your-app.com/oauth/authorize?'.$query);
});
```

### 2. Converting Authorization Codes To Access Tokens
If the user approves the authorization request, they will be redirected back to your application. You should then issue a POST request to request an access token. The request should include the authorization code that was issued in the callback from when the user approved the authorization request. In this example, we'll use the Guzzle HTTP library to make the POST request:

```php
Route::get('/callback', function (Request $request) {
    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'redirect_uri' => 'http://example.com/callback',
            'code' => $request->code,
        ],
    ]);

    return json_decode((string) $response->getBody(), true);
});
```

This `/oauth/token` route will return a JSON response containing `access_token`,  `refresh_token`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the access token expires.

```json
{
	"token_type": "Bearer",
	"expires_in": 1296000,
	"access_token": "EZfjfe34fe...",
	"refresh_token": "FEeafnfaef..."
}
```

### 3. Refreshing Tokens
A token lasts for 15 days, you will need to refresh access tokens via the refresh token that was provided together with the issued access token above. In this example, we'll use the Guzzle HTTP library to refresh the token:

```php
$http = new GuzzleHttp\Client;

$response = $http->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'refresh_token',
        'refresh_token' => 'the-refresh-token',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => '',
    ],
]);

return json_decode((string) $response->getBody(), true);
```
This `/oauth/token` route will return a JSON response containing `access_token`, `refresh_token`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the access token expires.


## Requesting User Data
Once you have acquired an `access_token`, you can use it to request the user to whom it belongs. This can be done by requesting `/oauth/user`

```php
$http = new GuzzleHttp\Client;

$response = $http->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'refresh_token',
        'refresh_token' => 'the-refresh-token',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => '',
    ],
    'headers' => [
	    'authorization' => 'Bearer {access_token}'
    ]
]);

return json_decode((string) $response->getBody(), true);
```
This will return an user object like the following

```json
{
	"id": 999,
	"email": "Alice@bob.dk",
	"locale": "da_dk",
	"brand_id": 1,
	"postal_code": null,
	"name": "Alice Mallory Trent",
	"first_name": "Alice Mallory",
	"last_name": "Trent",
	"username": "AliceT",
	"created_at": "2018-01-31 13:01:43",
	"subscription_number": "123456789",
	"roles": [
		"users", "subscribers"
	]
}
```

## Security and how to use state
Please carefully read this: https://auth0.com/docs/protocols/oauth2/oauth-state
