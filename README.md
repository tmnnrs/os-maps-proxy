# os-maps-proxy
Simple PHP script which can act as a proxy for making OS Maps tile requests while masking the API key; and restricting the origin to certain hosts.

```php
<?php

  $apikeys = array(
    'INSERT_API_KEY_HERE',
    ...
  );

  $in_array = array('Outdoor 3857', 'Road 3857', 'Light 3857', 'Night 3857');
  $out_array = array('Outdoor%203857', 'Road%203857', 'Light%203857', 'Night%203857');

  $request = apache_request_headers();

  if( $request['Host'] == 'example.com' || $request['Host'] == 'localhost:8888' ) {
    $mapping_api_url = $_GET['url'];
    $mapping_api_url = str_replace($in_array, $out_array, $mapping_api_url);

    $apikey = isset($_GET['id']) ? $apikeys[$_GET['id']] : $apikeys[0];

    $context = stream_context_create(array('http' => array('header'=>'Connection: close\r\n')));

    if( $image = @imagecreatefromstring(file_get_contents("$mapping_api_url?key=$apikey", false, $context)) ) {
      header('Content-Type: image/png');
      imagepng($image);
      imagedestroy($image);
    }
  }

?>
```

Requests can accept two parameters:

Name | Type | Required/Optional | Description
------------ | ------------ | ------------ | -------------
`id` | number | _optional_ | Array index. This allows provison for a project specific key. If omitted the default [0] key will be used.
`url` | string | _required_ | Declared map service.

Example request:

`/proxy.php?id=0&url=https://api2.ordnancesurvey.co.uk/mapping_api/v1/service/zxy/EPSG:3857/Light3857/{z}/{x}/{y}.png`
