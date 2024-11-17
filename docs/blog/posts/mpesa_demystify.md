---
date:
  created: 2018-10-15

authors:
  - kwahalf
categories:
  - Payment Integrations

comments: true

hide:
  - footer
---

# Demystifying The M-Pesa API (Lipa Na M-Pesa Online Payment)

![Image title](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*A4ovQ5AoCbbs1SXuGUD6tw.jpeg){ width="1400" }
/// caption
Payment matrix
///

[M-Pesa](https://en.wikipedia.org/wiki/M-Pesa) is a mobile phone-based money transfer, financing and microfinancing service, launched in 2007 by Vodafone for Safaricom and Vodacom the largest mobile network operators in Kenya and Tanzania.

<!-- more -->


M-Pesa allows users to deposit, withdraw, transfer money and pay for goods and services (Lipa na M-Pesa) easily with mobile devices. M-Pesa has spread quickly like wildfire, and by 2010 had become the most successful mobile-phone-based financial service in the world.

With M-Pesa winning the heart and souls of the masses, there was a section of the population who yearned for more. These were the software developers, they wanted an API to access the M-Pesa products. Safaricom responded to them by giving them the M-Pesa G2 API. This was welcomed but only a handful of developers were able to integrate their apps with it. Some did not have the infrastructure nor the technical know-how to do the integrations because of its requirements which included: VPN connection, soap protocol, etc.

Safaricom recently introduced a new M-Pesa API dubbed ‘[The Daraja API’](https://developer.safaricom.co.ke/) (Daraja in Swahili means bridge) to try and bridge this gap for developers. You can access this API over the public internet a step up from the G2 API which required a VPN connection. The other change was that it communicated over the REST protocol, this was to try and alleviate the pain of the developers whose Apps use REST protocol but had to find a way to accommodate M-Pesa’s SOAP. Daraja API not only had the old services from G2 API(C2B, B2C, and reversal) but also exposed new M-Pesa services(B2B, Transaction status, Account balance, and Lipa na M-Pesa online)

In this article, we’ll just cover the Lipa na M-Pesa online as this is one of the most exciting API they have provided so far.

## Prerequisites

1. You should have a Safaricom developers account. If you don’t have one go here [developers portal](https://developer.safaricom.co.ke/) and register to get an account. It is free.

2. I will be demonstrating the use of the API using Javascript hence you should at least have a grasp of the basics in javascript although you can follow and re-implement with the language of your choice.

3. You should have [Node](https://nodejs.org/en/) installed

4. We will be using [Ngrok](https://ngrok.com/) to expose our local server to the internet hence having it downloaded now is encouraged.

5. We will be using [Postman](https://www.getpostman.com/) to send requests so downloading it encouraged.

## Lipa Na M-Pesa Online

The Lipa na M-Pesa online service exposed by Daraja API is a very efficient way to implement merchant checkouts in e-commerce sites, mobile Apps, car parking management systems etc.

A common implementation is, you have a checkout button in your mobile App/ internet site which a user clicks to pay for the goods or services you are offering. This button when clicked triggers a post request to Daraja with the relevant payload. When Daraja receives the request it then triggers an STK push to the user’s phone prompting the user to enter his/her M-Pesa password to complete the transaction. When the user enters the password to complete the transaction, this request is sent to M-Pesa for processing, the user’s M-Pesa account is debited, and then you receive a webhook (to your servers webhook URL you provided) with the details of the transaction.

## Webhook Server

We will now implement the server which will be receiving these webhooks from Safaricom. Our server will have minimal functionality and nothing fancy hence you should not move with it to your production environment.

We will use ExpressJS a JS library to implement our simple server.

Okay let’s go to our terminal and create a new folder and call it webhook

``` sh
 mkdir webhook && cd webhook
```

Initiate our package.json in our directory

``` sh
npm init
```

Let’s now install our dependencies using npm

``` sh
npm i -s express body-parser prettyjson
```

we will use body-parser middleware to extract the entire body portion of an incoming request stream and exposes it on `req.body`.

we will use `prettyjson` to format the output that we will dump in the terminal.

Now let’s implement our server, create a file called `server.js`

``` js linenums="1"
  const prettyjson = require('prettyjson');
  const express = require('express');
  const bodyParser = require('body-parser')

  const options = {
    noColor: true
  };

  // create an express app and configure it with boadyParser middleware
  const app = express();
  app.use(bodyParser.json());
  app.use(bodyParser.urlencoded({ extended: true }));

  // create our webhook endpoint to recive webhooks from Safaricom
  app.post('/hooks/mpesa', (req, res) => {
    console.log('-----------Received M-Pesa webhook-----------');
    
    // format and dump the request payload recieved from safaricom in the terminal
    console.log(prettyjson.render(req.body, options));
    console.log('-----------------------');
    
    let message = {
      "ResponseCode": "00000000",
      "ResponseDesc": "success"
    };
    
    // respond to safaricom servers with a success message
    res.json(message);
  });

  const server = app.listen(5000, () => {
    let host = server.address().address;
    let port = server.address().port;
    console.log(`server listening on port ${port}` );
  });
```

Let's start our server and test it out

``` sh
node server.js
```

![Image title](https://miro.medium.com/v2/resize:fit:1222/format:webp/1*pXPPKRkfDQ3UtvcjmNRugg.png){ width="1222" }

Now that our server is running let's expose it to the internet using ngrok. Go to the folder where you downloaded and extracted ngrok and run

``` sh
./ngrok http 5000
```
![Image title](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Ykm2OPG8pO9_YjweBe11Ag.png){ width="1400" }


Now that we have connected our webhook server to the internet let’s test to see if we can reach it on the exposed URL. Fire up postman and send a JSON post to the link exposed on your ngrok followed by the endpoint of or webhook. for my case it will be `https://05135c4e.ngrok.io/hooks/mpesa` for you the `05135c4e.ngrok.io` will be different because every time you start ngrok it will give you a new unique URL.

![Image title](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*A7zlue0MqP7sh7IGLhgJvg.png){ width="2000" }

Our request is then dumped to the terminal as expected

![Image title](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*bZgl7tYNkYLr3Ivv0hYGQg.png){ width="1400" }

Now that we have developed our simple server and exposed it on the internet let us now interact with the M-Pesa API.

## M-Pesa App

Go to the [developers portal](https://developer.safaricom.co.ke/) log in with your credentials then click on the my apps tab to access your apps. Create a new app by clicking on the Add new App button which will take you to a page like the one bellow

![Image title](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*s6m-yBY9Z4aHRSNC96ec7g.png){ width="1400" }

Enter your App name and check the Lipa Na Mpesa Sandbox checkbox then click the create app button.

Click on the app you’ve created to see the details and the credentials you will use to access the M-Pesa API.

![Image title](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*m7bQvqFsTVvKfIwrXZXRrQ.png){ width="1400" }

!!! note
    I created this app for demonstration purposes and it was deleted after so the keys exposed are no longer functional for those of us who will be scrambling to use them.

Now that we have the credentials let’s authenticate ourselves and get a token that we will use to interact with the Lipa na M-Pesa service.

since we have not gone Live with our app we will use the sandbox endpoints to demonstrate (`https://sandbox.safaricom.co.ke/oauth/v1/generate?grant_type=client_credentials`). It uses basic auth to authenticate and returns our api token that we will use to access the Lipa na M-Pesa service. The username is your customer key and the password is your customer secret. So open postman and click the authorization tab and select type as basic auth. Fill in the Username and Password then paste the link url box.

![Image title](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*ywrv7ylZAM7OmVsyWzN-Sw.png){ width="2000" }

Then send the get request to get the token as shown bellow.

![Image title](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*8FDXwLeNNrglbpL9dcFpiw.png){ width="2000" }

Now that we have the access token let’s send our first request to the Lipa na M-Pesa service. This service expects a request payload that is JSON containing.:

1. BusinessShortCode: This is the organisation’s short-code (Paybill or Buygood) used to identify an organisation and receive the transaction.

2. Password: This is the password used for encrypting the request sent: A base64 encoded string. (The base64 string is a combination of `Short-code+Passkey+Timestamp`)

3. Timestamp: This is the Timestamp of the transaction, normally in the formart of `YEAR+MONTH+DATE+HOUR+MINUTE+SECOND` (YYYYMMDDHHMMSS) Each part should be at-least two digits apart from the year which takes four digits.

4. TransactionType: This is the transaction type that is used to identify the transaction when sending the request to M-Pesa. The transaction type for M-Pesa Express is “CustomerPayBillOnline”

5. Amount: This is the Amount transacted normally a numeric value. Money that customer pays to the Short-code. Only whole numbers are supported.

6. PartyA: The phone number sending money. The parameter expected is a Valid Safaricom Mobile Number that is M-Pesa registered in the format `2547XXXXXXXX`

7. PartyB: The organisation receiving the funds. The parameter expected is a 5 to 6 digit as defined on the Short-code description above. This can be the same as BusinessShortCode value above.

8. PhoneNumber: The Mobile Number to receive the STK Pin Prompt. This number can be the same as PartyA value above. Use only Safaricom numbers.

9. CallBackURL: A CallBack URL is a valid secure URL that is used to receive notifications from M-Pesa API. It is the endpoint to which the results will be sent by M-Pesa API.

10. AccountReference: This is an Alpha-Numeric parameter that is defined by your system as an Identifier of the transaction for CustomerPayBillOnline transaction type. Along with the business name, this value is also displayed to the customer in the STK Pin Prompt message. Maximum of 12 characters.

11. TransactionDesc: This is any additional information/comment that can be sent along with the request from your system. Maximum of 13 Characters

For testing we can get the testing credentials here once logged in.

On the credentials page we will use the Lipa Na Mpesa Online Short-code as our BusinessShortCode and PartyB. We will also use Lipa Na Mpesa Online Passkey as our Passkey for generating a base64 string password.

To generate your base64 string password go to this website and encode your

`Shortcode+Passkey+Timestamp` (eg 174379bfb279f9aa9bdbcf158e97dd71a467cd2e0c893059b10f78e6b72ada1ed2c91920181015123520) then copy the base64 encoded string to use it as your password.

We will then send a post request containing the payload described above and with an Authorization header containing:

`Bearer <Access-Token>` to this URL `https://sandbox.safaricom.co.ke/mpesa/stkpush/v1/processrequest`

So open postman and click the authorization tab and select type as Bearer token. Fill in the token (The API token we got from our authorization request)then paste the link url box.

![Image title](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*U-bCkoeuwJ7RVGkLJ3Df4A.png){ width="2000" }


Select the type of request to be a `post` request then click on the body tab, click on the raw payload, select the content type as `application/json` and enter the JSON payload and click send.

![Image title](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*iem2c1IH63ft1gEQtwAQ_w.png){ width="2000" }

The phone-number provided would then receive a STK push to enter the M-Pesa pin to complete the transaction.

![Image title](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*bBtn1u0Cs-XNxxVhW4-VPw.jpeg){ width="400" }

After the pin is entered the customer is debited then M-Pesa sends a webhook payload to the URL we provided.

![Image title](https://miro.medium.com/v2/resize:fit:1002/format:webp/1*jMtp-VvlGbi8zzNoNFuGCA.jpeg){ width="400" }

![Image title](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*PxHlyCKoaE8M8unkbRH_Lw.png){ width="2000" }

All the deducted funds during testing are automatically reversed to your account later by Safaricom.

## Conclusion

With this introduction you can continue to play around in the sandbox and explore this API. I hope the article was informative to you. You could leave comments on which M-Pesa API service you would like me to write an article on next. Until next time bye.

