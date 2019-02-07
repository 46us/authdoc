# helpappauth
Authorization plugin for WP Help App site.

# How does it work?
- User needs to login to the associated site, in this case is DWC product site.
- Inside product site, there should be a link which will redirect user to Help App site when user click it.
- Here is an example how to get Help App url per product per user session:
~~~
<?php
// Get current token per user session in our database.
$current_token = function_to_get_the_saved_token();

// If no current token available, then please set a random string
// so that the REST API url in the $token part is not empty.
// The $token part in REST API is required so that if the token is still valid and not yet expired,
// the Help App does not generate new token.
$token = ($current_token == '') ? 'empti' : $current_token;

// [product] should be replaced with our product name, for example "rst".
// "xxx" is the subsite name per product.
$help_app_url_per_product = 'http://xxx.newhelpstaging.rschooltoday.com/';
$rest_api_url = $help_app_url_per_product . 'wp-json/helpappauth/v1/url/[product]/' . $token;
$ch = curl_init($rest_api_url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);
$curl_result = curl_exec($ch);
curl_close($ch);

// The second parameter of json_decode is set to "true" so the result will be an associative array.
$json_data = json_decode($curl_result, true);

// The generated token below needs to be saved in our database to be used later,
// see $current_token part above. This token also will be used on logout process.
$new_token = $json_data['token'];
function_to_save_generate_token($new_token);

// The generated URL below will be used as Help App link's href.
$helpapp_url = $json_data['url'];
?>

<a href="<?php echo $helpapp_url; ?>" target="_blank">Help</a>
~~~
- The example above is using CURL, if you prefer another way, such as AJAX, then please do so accordingly.
- Here is a code example of logout process:
~~~
<?php
// Get current token per user session.
$current_token = function_to_get_current_token_per_user_session();
$token = ($current_token == '') ? 'empti' : $current_token;

// [product] should be replaced with our product name, for example "rst".
$help_app_url_per_product = 'http://xxx.newhelpstaging.rschooltoday.com/';
$rest_api_url = $help_app_url_per_product . 'wp-json/helpappauth/v1/logout/[product]/' . $token;

// The logout process.
$ch = curl_init($rest_api_url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);
$curl_result = curl_exec($ch);
curl_close($ch);

// Delete current token per user session from database.
function_to_delete_current_token();
~~~
- Put the code above on your product site's logout process.
