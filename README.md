<p align="center">
  <a href="https://laravel.com">
    <img src="https://laravel.com/img/logomark.min.svg" alt="Logo" height="80">
  </a>
  <img src="https://img.icons8.com/material-outlined/96/000000/plus-math--v1.png"/ height="30">
  <a href="https://zibal.ir/">
    <img src="https://zibal.ir/static/media/logo-primary.68b6aace.svg" alt="Logo" height="80">
  </a>

  
  <br />
  <h2 align="center">laravel zibal</h2>

  <p align="center">
    transaction request package for <a href="https://zibal.ir/">zibal</a>
    <br />
  </p>
</p>






<!-- GETTING STARTED -->
## Getting Started

To get a local copy up and running follow these simple steps.

### Installation

1. You can install the package via composer:
   ```sh
   composer require llabbasmkhll/laravel-zibal 
   ```
   
note that you only need to do following steps if you want to change merchant id . if you only want to test the webservice , no need to do these steps

2. publish config file to your project
   ```sh
   php artisan vendor:publish
   ```
3. change merchant value to your merchant id in config/zibal.php ( use `zibal` for testing )
   ```php
   return [
         'merchant' => 'zibal',
   ];
   ```


<!-- USAGE EXAMPLES -->
## Usage
according to zibals [official documentation](https://docs.zibal.ir/IPG/API)
there is 3 steps to issue a transaction in zibal

#### 1 . Request
  in this step zibal gets basic information about the transaction and returns `trackId` that is needed for next step 
#### 2 . Start 
  redirect the user to zibals gateway for payment
#### 3 . Verify
  you can inspect the status of the transaction in this step (either successful or not)

<br />

### code snippets : 
first include package facade into your file by :
```php
use Llabbasmkhll\LaravelZibal\Facades\Zibal;
```

use this to init the transaction request (first step) :

```php


Zibal::init(
    1000000,            //required amount          - in rial
    'redirect',         //required callback        - can be either route name or a valid url starting with http or https
    ['key' => 'value'], //optional callback_params - will be passed to callback , works only when route name passed to callback
    'description',      //optional description     - additional data , good for various reports
    123,                //optional orderId         - id of clients order (eg $invoice->id) , will be passed back to callback
    '09366217515',      //optional mobile          - clients mobile number
    ['000000000000']    //optional allowedCards    - array of allowed card numbers
)->getResponse();


```
this will give you an array consist of `result` , `message` and `trackId`

`result` represents the request status as below.
| status | meaning    |
|---------|---------------|
| 100    | successful operation |
| 102    | merchant not found |
| 103    | merchant not active |
| 104    | merchant not valid |
| 201    | processed before |
| 105    | amount must be grater than 1000 |
| 106    | callbackUrl is not valid |
| 113    | amount greater than maximum |

you can add `validate()` function after init like below :
```php

Zibal::init( $amount, 'redirect')->validate(404)->getResponse();

```

this will redirect the user to 404 page if the result code was anything except 100. 

<br />

to redirecting the user to zibals payment page (second step) use  :
   ```php
   Zibal::redirect($trackId);
   ```
   
<br />

you may combine first and second step into one line of code like this :
   ```php
   Zibal::init( $amount, 'redirect')->validate()->redirect();
   ```
   
<br />

finally you can use this line of code to verify the transaction status (third step):
   ```php
   Zibal::verify($trackId)->validate()->getResponse();
   ```
   
<br />

<br />

### example controller : 

```php

<?php

namespace App\Http\Controllers\Web;

use App\Http\Controllers\Controller;
use App\Models\Invoice;
use Illuminate\Http\Request;
use Llabbasmkhll\LaravelZibal\Facades\Zibal;

class GatewayController extends Controller
{

    /**
     * redirect user to zibal.
     *
     * @param  \Illuminate\Http\Request  $request
     *
     * @return \Illuminate\Contracts\Foundation\Application|\Illuminate\Http\RedirectResponse|\Illuminate\Routing\Redirector
     */
    public function show(Request $request)
    {
        $invoice = Invoice::find($request->input('invoice'));
        $user    = $invoice->user()->first();


        return Zibal::init(
            $invoice->amount,
            'redirect',
            [],
            $user->id,
            $invoice->id,
            $user->mobile,
        )->validate()->redirect();
    }


    /**
     * validate the transaction.
     *
     * @return \Illuminate\Contracts\Foundation\Application|\Illuminate\Http\RedirectResponse|\Illuminate\Routing\Redirector
     */
    public function index(Request $request)
    {
        abort_unless($request->has('trackId'), 422);

        $response = Zibal::verify($request->input('trackId'))->getResponse();

        if ($response['result'] == 100) {
            $invoice = Invoice::find($response['orderId']);
            $invoice->update(
                [
                    'status'       => 1,
                    'bank_code'    => $response['refNumber'],
                    'finalized_at' => now(),
                ]
            );
        }

        return redirect('panel');
    }

}

```


<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

<br />

<br />

<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE` for more information.

<br />

<br />

<!-- CONTACT -->
## Contact

Abbas mkhzomi - [Telegram@llabbasmkhll](https://t.me/llabbasmkhll) - llabbasmkhll@gmail.com

Project Link: [https://github.com/llabbasmkhll/laravel-zibal](https://github.com/llabbasmkhll/laravel-zibal)


