# StripeIntegration
Integrating Stripe in an iOS app using Heroku and Swift
 
 # Downloads Necessary
 Download Composer:  https://getcomposer.org/download/
 Download Git: https://git-scm.com/downloads
 Download PHP: http://php-osx.liip.ch/
 Download Heroku Toolbelt: https://toolbelt.heroku.com/
 Also need Cocoapods and Xcode
 
 # Setting Up The Server
 open terminal
 run:
 ```
 cd
 mkdir stripe-integration
 ```
 open Xcode 
 -> create a single view application 
 -> name it "stripe-test" or replace any sections of the following code where stripe-test is used with your project name
 -> save it to the stripe-integration directory
 back to terminal: 
 ```
 cd
 cd stripe-integration
 mkdir stripe-server
 cd
 clear
 cd
 cd stripe-integration
 cd stripe-server
 git init
 php composer.phar require stripe/stripe-php
 touch Procfile
 touch index.php
 touch charge.php
 open Procfile
 open index.php
 open charge.php
 cd
 clear
 ```
 copying that code into terminal should open the Procfile, index.php, charge.php.
 in the Procfile paste:
 ```
 web: vendor/bin/heroku-php-apache2
 ```
 in index.php paste (this code is not necessary but is a good check of whether or not the server is working)
 ```
 <html>
<body>
<p>
<?php
echo “Hello world!”;
?>
</p>
</body>
</html>
 ```
 in the charge.php file paste:
 ```
 <?php
require_once(‘vendor/autoload.php’);
```
below, where it says "stripe test key" input your stripe test publishable key that you can get 
by going to your stripe dashboard and clicking on API on the left side. 
```
\Stripe\Stripe::setApiKey(“stripe test key”);
$token = $_POST[‘stripeToken’];
$amount = $_POST[‘amount’];
$currency = $_POST[‘currency’];
$description = $_POST[‘description’];
try {
 $charge = \Stripe\Charge::create(array(
 “amount” => $amount*100, // Convert amount in cents to dollar
 “currency” => $currency,
 “source” => $token,
 “description” => $description)
 );
// Check that it was paid:
 if ($charge->paid == true) {
 $response = array( ‘status’=> ‘Success’, ‘message’=>’Payment has been charged!!’ );
 } else { // Charge was not paid!
 $response = array( ‘status’=> ‘Failure’, ‘message’=>’Your payment could NOT be processed because the payment system rejected the transaction. You can try again or use another card.’ );
 }
 header(‘Content-Type: application/json’);
 echo json_encode($response);
} catch(\Stripe\Error\Card $e) {
 // The card has been declined
header(‘Content-Type: application/json’);
 echo json_encode($response);
}
?>
 ```
 Save all of the files and close them.
 back to terminal:
 ```
 cd
 cd stripe-integration
 cd stripe-server
 git add .
 git commit -m "First Upload"
 heroku create
 git push heroku master
 heroku open
 cd
 clear
 cd stripe-integration
 cd stripe-test
 pod init
 open -a Xcode Podfile
 cd
 ```
 your app's podfile should now be opened.
 under #pods for "project-name" paste:
 ```
 pod 'Stripe'
 pod 'Alamofire'
 ```
 back to terminal:
 ```
 cd stripe-integration
 cd stripe-test
 pod install
 open stripe-test.xcworkspace
 ```
 this will prep you for the next part
 
 # Integrating The Stripe SDK in Xcode
 First we need a bridging header because stripe is written in Obj-C:
 Click Main.storyboard (just to make sure we are in the correct folder)
 File -> new -> File... -> Header File
 name the file "bridging-header" and save it
 navigate to Build Settings and search Bridging Header
 in the text field for "Objective-C Bridging Header" type "bridging-header.h"
 go to the bridging-header.h file 
 after #define and before #endif paste:
 ```
 #import <Stripe/Stripe.h>
 ```
 now go to appDelegate.swift and change the application didFinishLaunchingWithOptions function to include the following line:
 (make sure you replace the test publishable key with your own)
 ```
 Stripe.setDefaultPublishableKey("pk_test_publishableKey")
 ```
 
 # Adding UI Elements
 go to Main.storyboard and add a button and change its label to "Pay".
 open up a dual view with the storyboard on one side and ViewController.swift on the other side
 connect the pay button to an outlet in ViewController.swift name the outlet payButtonOutlet
 connect the pay button to an action in ViewController.swift name the action payButtonAction
 change the class header to include STPPaymentCardTextFieldDelegate: 
 (this will cause an error temporarily)
 ```
 class ViewController: UIViewController, STPPaymentCardTextFieldDelegate
 ```
 create a constant at the top to hold the stripe payment textfield:
 ```
 let paymentTextField = STPPaymentCardTextField()
 ```
 add this code to the function viewDidLoad():
 ```
paymentTextField.frame = CGRect(x: 15 , y: 199, width: self.view.frame.width - 30 , height: 44)
paymentTextField.delegate = self
view.addSubview(paymentTextField)
self.payButtonOutlet.isHidden = true
 ```
 Now add the function paymentCardTextFieldDidChange to conform to the STPPaymentCardTextFieldDelegate protocol:
 (also add the if statement inside of it)
 ```
 func paymentCardTextFieldDidChange(textField: STPPaymentCardTextField) { 
  if textField.valid {
      pay.hidden = false;
   }
}
 ```
 Now make a function to make the charge on our heroku server:
 (below: change the request string to your charge.php link. Don't know how to get that? scroll to the bottom of the README.)
 ```
     func chargeUsingToken(token: STPToken){
        let requestString = " "
        let params = ["stripeToken": token.tokenId, "amount": "200", "currency": "usd", "description": "testRun"]
        
        Alamofire.request(requestString , method: .post , parameters: params).responseJSON { response in
                        print(response.request)
                        print(response.response)
                        print(response.data)
                        print(response.result)
            
                        if let JSON = response.result.value {
                            print("JSON: \(JSON)")
                        }
                    }

        
    }
 ```
 Let's go back to the function payButtonAction, add the following code:
 ```
         let card = paymentTextField.cardParams
        STPAPIClient.shared().createToken(withCard: card, completion: {(token, error) -> Void in
            if let error = error {
                print(error)
            }
            else if let token = token {
                print(token)
                self.chargeUsingToken(token: token)
            }
        })
```

# Getting the charge.php link
go to terminal:
```
cd 
cd stripe-integration
cd stripe-server
heroku open
```
copy the link in the address bar
add to that link: /charge.php
it should look something like this: https://<...>.herokuapp.com/charge.php

 
