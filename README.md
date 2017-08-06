globalcash Instant Payment Notifications (IPN)
Instant Payment Notifications (IPN)
Introduction
IPN Setup
IPN Retries
Authenticating IPNs
Payment Statuses
Code Samples
IPN POST Fields
Introduction
The IPN system will notify your server when you receive a payment and when a payment status changes. This is a easy and useful way to integrate our payments into your software to automate order completion, digital downloads, accounting, or whatever you can think up.
It is implemented by making a HTTP POST call over a https:// or http:// URL to a script or CGI program on your server.
IPN Setup
The first step is to go to the My Settings page and set a IPN Secret. Your IPN Secret is a string of your choosing that is used to verify that an IPN was really sent from our servers (recommended to be a random string of letters, numbers, and special characters). Our system will not send any IPNs unless you have an IPN Secret set. See the "Authenticating IPNs" section for more details.

At the same time you can optionally set an IPN URL; this is the URL that will be called when sending you IPN notifications. You can also set an IPN URL in your Buy Now and Cart buttons that will be used instead of this one.
IPN Retries
If there is an error sending your server an IPN, we will retry up to 5 times. Because of this you are not guaranteed to receive every IPN (if all 5 tries fail) or that your server will receive them in order.
Authenticating IPNs
There are two options for verifying IPNs: HTTP Auth and HMAC. HTTP Auth is compatible with more server types and languages, but HMAC is more secure.
By default we will use the HTTP Auth method.
Method: HTTP Auth

The IPN will pass a HTTP Basic Authentication username and password to your IPN handler.
Username = Your Merchant ID
Password = Your IPN Secret
As a quick example, it would look like this in PHP:
if (isset($_SERVER['PHP_AUTH_USER']) && isset($_SERVER['PHP_AUTH_PW'])) {
  if ($_SERVER['PHP_AUTH_USER'] == "Your_Merchant_ID" && $_SERVER['PHP_AUTH_PW'] == "Your_IPN_Secret") {
    //process IPN here
  }
}
Method: HMAC

In this method we use your IPN Secret as the HMAC shared secret key. The HMAC signature is sent as a HTTP header called HMAC.
Here is what it would look like in PHP:
$merchant_id = 'Your_Merchant_ID';
$secret = 'Your_IPN_Secret';

if (!isset($_SERVER['HTTP_HMAC']) || empty($_SERVER['HTTP_HMAC'])) {
  die("No HMAC signature sent");
}

$request = file_get_contents('php://input');
if ($request === FALSE || empty($request)) {
  die("Error reading POST data");
}

$merchant = isset($_POST['merchant']) ? $_POST['merchant']:'';
if (empty($merchant)) {
  die("No Merchant ID passed");
}
if ($merchant != $merchant_id) {
  die("Invalid Merchant ID");
}

$hmac = hash_hmac("sha512", $request, $secret);
if ($hmac != $_SERVER['HTTP_HMAC']) {
  die("HMAC signature does not match");
}

//process IPN here
Payment Statuses
Payments will post with a 'status' field, here are the currently defined values:
-2 = PayPal Refund or Reversal
-1 = Cancelled / Timed Out
0 = Waiting for buyer funds
1 = We have confirmed coin reception from the buyer
2 = Queued for nightly payout (if you have the Payout Mode for this coin set to Nightly)
3 = PayPal Pending (eChecks or other types of holds)
100 = Payment Complete. We have sent your coins to your payment address or 3rd party payment system reports the payment complete
For future-proofing your IPN handler you can use the following rules:
<0 = Failures/Errors
0-99 = Payment is Pending in some way
>=100 = Payment completed successfully
IMPORTANT: You should never ship/release your product until the status is >= 100 OR == 2 (Queued for nightly payout)!
Code Samples
A complete example of an IPN Handler can be found in: [PHP].
Example building Ripple transactions in response to IPNs with Quick Gateway Kit: https://github.com/whotooktwarden/rippled-sign-submit.
Handling IPNs for deposits with Quick Gateway Kit: https://github.com/whotooktwarden/QuickGatewayKit.
IPN POST Fields
Field Name	Description	Required?
Required Fields
These fields will be here for all IPN types.
ipn_version	1.0	Yes
ipn_type	Currently: 'simple, 'button', 'cart', 'donation', 'deposit', or 'api'	Yes
ipn_mode	Currently: 'httpauth' or 'hmac'	Yes
ipn_id	The unique identifier of this IPN	Yes
merchant	Your merchant ID (you can find this on the My Account page).	Yes
Deposit Information (ipn_type = 'deposit')
address	Coin address the payment was received on.	Yes
txn_id	The coin transaction ID of the payment.	Yes
status	Numeric status of the payment, currently 0 = pending and 100 = confirmed/complete. For future proofing you should use the same logic as Payment Statuses.
IMPORTANT: You should never ship/release your product until the status is >= 100	Yes
status_text	A text string describing the status of the payment. (useful for displaying in order comments)	Yes
currency	The coin the buyer paid with.	Yes
confirms	The number of confirms the payment has.	Yes
amount	The total amount of the payment	Yes
amounti	The total amount of the payment in Satoshis	Yes
fee	The fee deducted by CoinPayments (only sent when status >= 100)	No
feei	The fee deducted by CoinPayments in Satoshis (only sent when status >= 100)	No
Withdrawal Information (ipn_type = 'withdrawal')
id	The ID of the withdrawal ('id' field returned from 'create_withdrawal'.)	Yes
status	Numeric status of the withdrawal, currently 0 = waiting email confirmation, 1 = pending, and 2 = sent/complete.	Yes
status_text	A text string describing the status of the withdrawal.	Yes
address	Coin address the withdrawal was sent to.	Yes
txn_id	The coin transaction ID of the withdrawal.	No
currency	The coin of the withdrawal.	Yes
amount	The total amount of the withdrawal	Yes
amounti	The total amount of the withdrawal in Satoshis	Yes
Buyer Information (ipn_type = 'simple','button','cart','donation')
first_name	Buyer's first name.
Note: Only first_name or last_name is required, either may be empty but not both.	Yes
last_name	Buyer's last name.
Note: Only first_name or last_name is required, either may be empty but not both.	Yes
company	Buyer's company name.	No
email	Buyer's email address.	Yes
Shipping Information (ipn_type = 'simple','button','cart','donation')
If 'want_shipping' was set to 1 we will collect and forward shipping information, but as always they could have manually messed with your button so be sure to verify it.
address1	Street / address line 1	No
address2	Street / address line 2	No
city	City	No
state	State / Province	No
zip	Zip / Postal Code	No
country	Country of Residence
This uses 2 digit ISO 3166 country codes.	No
country_name	Country of Residence
This is a pretty version such as UNITED STATES or CANADA.	No
phone	Phone Number	No
Simple Button Fields (ipn_type = 'simple')
status	The status of the payment. See Payment Statuses for details.	Yes
status_text	A text string describing the status of the payment. (useful for displaying in order comments)	Yes
txn_id	The unique ID of the payment.
Your IPN handler should be able to handle a txn_id composed of any combination of a-z, A-Z, 0-9, and - up to 128 characters long for future proofing.	Yes
currency1	The original currency/coin submitted in your button.
Note: Make sure you check this, a malicious user could have changed it manually.	Yes
currency2	The coin the buyer chose to pay with.	Yes
amount1	The total amount of the payment in your original currency/coin.	Yes
amount2	The total amount of the payment in the buyer's selected coin.	Yes
subtotal	The subtotal of the order before shipping and tax in your original currency/coin.	Yes
shipping	The shipping charged on the order in your original currency/coin.	Yes
tax	The tax on the order in your original currency/coin.	Yes
fee	The fee on the payment in the buyer's selected coin.	Yes
net	The net amount you received of the buyer's selected coin after our fee and any coin TX fees to send the coins to you.	Yes
item_amount	The amount of the item/order in the original currency/coin.	Yes
item_name	The name of the item that was purchased.	Yes
item_desc	Description of the item that was purchased.	No
item_number	This is a passthru variable for your own use. [visible to buyer]	No
invoice	This is a passthru variable for your own use. [not visible to buyer]	No
custom	This is a passthru variable for your own use. [not visible to buyer]	No
on1	1st option name. This lets you pass through a buyer option like size or color.	No
(unless ov1 set)
ov1	1st option value. This would be the buyer's selection such as small, large, red, white.	No
on2	2nd option name. This lets you pass through a buyer option like size or color.	No
(unless ov2 set)
ov2	2nd option value. This would be the buyer's selection such as small, large, red, white.	No
send_tx	The TX ID of the payment to the merchant. Only included when 'status' >= 100 and if the payment mode is set to ASAP or Nightly or if the payment is PayPal Passthru.	No
received_amount	The amount of currency2 received at the time the IPN was generated.	No
received_confirms	The number of confirms of 'received_amount' at the time the IPN was generated.	No
Advanced Button Fields (ipn_type = 'button')
status	The status of the payment. See Payment Statuses for details.	Yes
status_text	A text string describing the status of the payment. (useful for displaying in order comments)	Yes
txn_id	The unique ID of the payment.
Your IPN handler should be able to handle a txn_id composed of any combination of a-z, A-Z, 0-9, and - up to 128 characters long for future proofing.	Yes
currency1	The original currency/coin submitted in your button.
Note: Make sure you check this, a malicious user could have changed it manually.	Yes
currency2	The coin the buyer chose to pay with.	Yes
amount1	The total amount of the payment in your original currency/coin.	Yes
amount2	The total amount of the payment in the buyer's selected coin.	Yes
subtotal	The subtotal of the order before shipping and tax in your original currency/coin.	Yes
shipping	The shipping charged on the order in your original currency/coin.	Yes
tax	The tax on the order in your original currency/coin.	Yes
fee	The fee on the payment in the buyer's selected coin.	Yes
net	The net amount you received of the buyer's selected coin after our fee and any coin TX fees to send the coins to you.	Yes
item_amount	The amount per-item in the original currency/coin.	Yes
item_name	The name of the item that was purchased.	Yes
quantity	The quantity of items bought.	Yes
item_number	This is a passthru variable for your own use. [visible to buyer]	No
invoice	This is a passthru variable for your own use. [not visible to buyer]	No
custom	This is a passthru variable for your own use. [not visible to buyer]	No
on1	1st option name. This lets you pass through a buyer option like size or color.	No
(unless ov1 set)
ov1	1st option value. This would be the buyer's selection such as small, large, red, white.	No
on2	2nd option name. This lets you pass through a buyer option like size or color.	No
(unless ov2 set)
ov2	2nd option value. This would be the buyer's selection such as small, large, red, white.	No
extra	A note from the buyer.	No
send_tx	The TX ID of the payment to the merchant. Only included when 'status' >= 100 and if the payment mode is set to ASAP or Nightly or if the payment is PayPal Passthru.	No
received_amount	The amount of currency2 received at the time the IPN was generated.	No
received_confirms	The number of confirms of 'received_amount' at the time the IPN was generated.	No
Shopping Cart Button Fields (ipn_type = 'cart')
status	The status of the payment. See Payment Statuses for details.	Yes
status_text	A text string describing the status of the payment. (useful for displaying in order comments)	Yes
txn_id	The unique ID of the payment.
Your IPN handler should be able to handle a txn_id composed of any combination of a-z, A-Z, 0-9, and - up to 128 characters long for future proofing.	Yes
currency1	The original currency/coin submitted in your button.
Note: Make sure you check this, a malicious user could have changed it manually.	Yes
currency2	The coin the buyer chose to pay with.	Yes
amount1	The total amount of the payment in your original currency/coin.	Yes
amount2	The total amount of the payment in the buyer's selected coin.	Yes
subtotal	The subtotal of the order before shipping and tax in your original currency/coin.	Yes
shipping	The shipping charged on the order in your original currency/coin.	Yes
tax	The tax on the order in your original currency/coin.	Yes
fee	The fee on the payment in the buyer's selected coin.	Yes
item_name_#	The name of the item that was purchased. The # starts with 1.	Yes
item_amount_#	The amount per-item in the original currency/coin.	Yes
item_quantity_#	The quantity of items bought.	Yes
item_number_#	This is a passthru variable for your own use. [visible to buyer]	No
item_on1_#	1st option name. This lets you pass through a buyer option like size or color.	No
(unless ov1 set)
item_ov1_#	1st option value. This would be the buyer's selection such as small, large, red, white.	No
item_on2_#	2nd option name. This lets you pass through a buyer option like size or color.	No
(unless ov2 set)
item_ov2_#	2nd option value. This would be the buyer's selection such as small, large, red, white.	No
invoice	This is a passthru variable for your own use. [not visible to buyer]	No
custom	This is a passthru variable for your own use. [not visible to buyer]	No
extra	A note from the buyer.	No
send_tx	The TX ID of the payment to the merchant. Only included when 'status' >= 100 and if the payment mode is set to ASAP or Nightly or if the payment is PayPal Passthru.	No
received_amount	The amount of currency2 received at the time the IPN was generated.	No
received_confirms	The number of confirms of 'received_amount' at the time the IPN was generated.	No
Donation Button Fields (ipn_type = 'donation')
status	The status of the payment. See Payment Statuses for details.	Yes
status_text	A text string describing the status of the payment. (useful for displaying in order comments)	Yes
txn_id	The unique ID of the payment.
Your IPN handler should be able to handle a txn_id composed of any combination of a-z, A-Z, 0-9, and - up to 128 characters long for future proofing.	Yes
currency1	The original currency/coin submitted in your button.
Note: Make sure you check this!	Yes
currency2	The coin the donator chose to pay with.	Yes
amount1	The total amount of the payment in your original currency/coin.	Yes
amount2	The total amount of the payment in the donator's selected coin.	Yes
subtotal	The subtotal of the order before shipping and tax in your original currency/coin.	Yes
shipping	The shipping charged on the order in your original currency/coin.	Yes
tax	The tax on the order in your original currency/coin.	Yes
fee	The fee on the payment in the donator's selected coin.	Yes
net	The net amount you received of the buyer's selected coin after our fee and any coin TX fees to send the coins to you.	Yes
item_name	The name of the donation.	Yes
item_number	This is a passthru variable for your own use. [not visible to donator]	No
invoice	This is a passthru variable for your own use. [not visible to donator]	No
custom	This is a passthru variable for your own use. [not visible to donator]	No
on1	1st option name. This lets you pass through a donator option like size or color.	No
(unless ov1 set)
ov1	1st option value. This would be the donator's selection such as small, large, red, white.	No
on2	2nd option name. This lets you pass through a donator option like size or color.	No
(unless ov2 set)
ov2	2nd option value. This would be the donator's selection such as small, large, red, white.	No
extra	A note from the donator.	No
send_tx	The TX ID of the payment to the merchant. Only included when 'status' >= 100 and if the payment mode is set to ASAP or Nightly or if the payment is PayPal Passthru.	No
received_amount	The amount of currency2 received at the time the IPN was generated.	No
received_confirms	The number of confirms of 'received_amount' at the time the IPN was generated.	No
API Generated Transaction Fields (ipn_type = 'api')
status	The status of the payment. See Payment Statuses for details.	Yes
status_text	A text string describing the status of the payment. (useful for displaying in order comments)	Yes
txn_id	The unique ID of the payment.
Your IPN handler should be able to handle a txn_id composed of any combination of a-z, A-Z, 0-9, and - up to 128 characters long for future proofing.	Yes
currency1	The original currency/coin submitted.	Yes
currency2	The coin the buyer paid with.	Yes
amount1	The amount of the payment in your original currency/coin.	Yes
amount2	The amount of the payment in the buyer's coin.	Yes
fee	The fee on the payment in the buyer's selected coin.	Yes
buyer_name	The name of the buyer.	No
item_name	The name of the item that was purchased.	No
item_number	This is a passthru variable for your own use.	No
invoice	This is a passthru variable for your own use.	No
custom	This is a passthru variable for your own use.	No
send_tx	The TX ID of the payment to the merchant. Only included when 'status' >= 100 and if the payment mode is set to ASAP or Nightly or if the payment is PayPal Passthru.	No
received_amount	The amount of currency2 received at the time the IPN was generated.	No
received_confirms	The number of confirms of 'received_amount' at the time the IPN was generated.	No
