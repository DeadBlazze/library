```php
public function handle(Request $request, Closure $next): Response  
{  
	$response = $next($request);  
	$response->headers->set('Access-Control-Allow-Origin', '*'); // Разрешить запросы со всех доменов  
	$response->headers->set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');  
	$response->headers->set('Access-Control-Allow-Headers', 'Content-Type, Authorization');  
	return $response;  
}

public function handle(Request $request, Closure $next): Response
{  
	try{
		$token = JWTAuth::parseToken();
		$payload = $token->getPayload();
	} catch (\Exception $e) {
		return response()->json(['err' => 'Ошибка авторизации: ' . $e->getMessage()], 401);
	}
	$request->attributes->add(["payload"=> $payload]);
	return $next($request);
}

$id_user = DB::getPdo()->lastInsertId();
$payloadParam = [
	'sub' => $id_user,
	'iat' => now()->timestamp,
	'exp' => now()->addDays(3)->timestamp,
	'roles' => $roles
];
$payload = JWTFactory::customClaims($payloadParam)->make();
$token = JWTAuth::encode($payload)->get();

$payload = $request->attributes->get('payload');
$role = $payload->get('roles');

$validator = Validator::make($request->all(), [
            'tel' => ['required', 'regex:/^(\+7|8)\s?\d{3}\s?\d{3}\s?\d{2}\s?\d{2}$/'],
]);
```