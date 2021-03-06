# Error codes for Namebase
Errors consist of two parts: an error code and a message. Codes are universal,
 but messages can vary. Here is the error JSON payload:
```javascript
{
  "code": "NAMEBASE_API_ERROR_CODE",
  "message": "Error message appears here."
}
```

## General server or network issues; unclear fix
#### SERVER_UNKNOWN
* An unknown server error occured.

#### SERVER_FUTURE_TIMESTAMP
* The provided timestamp is too far into the future.

#### SERVER_LATE_TIMESTAMP
* This request was received after the specified receive window.

#### SERVER_MAINTENANCE
* The server is currently undergoing scheduled maintenance. Try again.

## Request issues; fixed by changing a parameter

#### REQUEST_UNAUTHENCIATED
* This request is not properly authenticated. Log in or use an API key.

#### REQUEST_PARSE_ERROR
* An unknown error occured while parsing your request parameters.

#### REQUEST_MISSING_PARAMETER
* Parameter ${key} is required but did not appear in the request.

#### REQUEST_BAD_PARAMETER
* Parameter ${key} does not adhere to the spec.

#### REQUEST_EXTRA_PARAMETER
* Parameter ${key} should not have been included.

#### REQUEST_UNSUPPORTED_SYMBOL
* The Namebase exchange does not yet support symbol "${symbol}".

#### REQUEST_UNSUPPORTED_ASSET
* The Namebase exchange does not yet support asset "${asset}".

#### REQUEST_UNSUPPORTED_LIMIT
* The only supported limits are ${acceptedLimitsString}.

#### REQUEST_MINIMUM_WITHDRAWAL
* The minimum withdrawal for ${asset} is ${amount}.

#### REQUEST_MINIMUM_DEPOSIT
* The minimum deposit for ${asset} is ${amount}.

#### REQUEST_MINIMUM_ORDER
* The minimum order size for ${symbol} is ${amount} ${asset}.

## Stateful errors; hard or impossible for user to fix
#### NOT_ALLOWED_TO_TRADE
* Your account is not currently allowed to trade. Complete KYC verification or contact support.

#### NOT_ALLOWED_TO_DEPOSIT
* Unfortunately, based on your verification status, you cannot deposit ${asset}.

#### NOT_ALLOWED_TO_WITHDRAW
* Unfortunately, based on your verification status, you cannot withdraw ${asset}.

#### WITHDRAWAL_ADDRESS_INVALID
* The withdrawal address you've entered is not a valid ${asset} address.

#### WITHDRAWAL_LIMIT_REACHED
* You've reached your withdrawal limit for ${asset}. Try withdrawing less funds or try again later.

#### WITHDRAWALS_DISABLED
* ${asset} withdrawals are disabled until block height ${height}.

#### INSUFFICIENT_BALANCE
* You do not have sufficient balance to perform this action.

#### CANCEL_INVALID
You can only cancel active orders.

#### CANCEL_UNAUTHORIZED
You do not have permission to cancel this order.

#### CANCEL_UNKNOWN
Could not cancel this order due to an unknown error.

#### GET_ORDER_FAILED
Could not retrieve this order.

#### GET_TRADES_UNAUTHORIZED
You do not have permission to retrieve this order's trades.
