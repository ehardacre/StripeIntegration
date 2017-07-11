# StripeIntegration
Integrating Stripe in an iOS app using Heroku and Swift
 
 # Downloads Necessary
 Download Composer: <br />
 Download Git: https://git-scm.com/downloads <br />
 Download PHP: http://php-osx.liip.ch/ <br />
 Download Heroku Toolbelt (you will need homebrew installed to download this): <br />
 in terminal run:
 ```
 brew install heroku
 ```
 <br />
 Also need Cocoapods and Xcode
 
 # Setting Up The Server
 open terminal <br />
 run:
 ```
 cd
 mkdir stripe-integration
 ```
 open Xcode <br />
 -> create a single view application <br />
 -> name it "stripe-test" or replace any sections of the following code where stripe-test is used with your project name <br />
 -> save it to the stripe-integration directory <br />
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
 php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
 php -r "if (hash_file('SHA384', 'composer-setup.php') === '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
 php composer-setup.php
 php -r "unlink('composer-setup.php');"
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
 copying that code into terminal should open the Procfile, index.php, charge.php. <br />
 in the Procfile paste:
 ```
 web: vendor/bin/heroku-php-apache2
 ```
 in index.php paste (this code is not necessary but is a good check of whether or not the server is working). (Seems to work better if quotes are vertical not curled).
 ```
<html>
 <body>
 <?php echo "<p>Hello World</p>"; ?> 
 </body>
</html>
 ```
 in the charge.php file paste:
 (Again, seems to work better with vertical quotes rather than curled. In other words, make sure strings are recognized as strings.)
 ```
 <?php
require_once('vendor/autoload.php');
```
below, where it says "stripe test key" input your stripe test PRIVATE key that you can get <br />
by going to your stripe dashboard and clicking on API on the left side. 
```
  \Stripe\Stripe::setApiKey("sk_test_yourprivatetestkey");
  $token = $_POST['stripeToken'];
  $amount = $_POST['amount'];
  $currency = $_POST['currency'];
  $description = $_POST['description'];
  try {
   $charge = \Stripe\Charge::create(array(
   "amount" => $amount*100, // Convert amount in cents to dollar
   "currency" => $currency,
   "source" => $token,
   "description" => $description)
   );
  // Check that it was paid:
   if ($charge->paid == true) {
   $response = array("status"=>"Success","message"=>"Payment has been charged!!");
   } else { // Charge was not paid!
   $response = array( 'status'=> 'Failure', 'message'=>'Your_payment_could NOT be processed because the payment system rejected the transaction. You can try again or use another card.' );
   }
   header('Content-Type: application/json');
   echo json_encode($response);
  } catch(\Stripe\Error\Card $e) {
   // The card has been declined
  header('Content-Type: application/json');
   echo json_encode($response);
  }
?>
 ```
 Save all of the files and close them. <br />
 back to terminal:
 ```
 cd
 cd stripe-integration
 cd stripe-server
 git add .
 git commit -m "First Upload"
 heroku create
 heroku login
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
 your app's podfile should now be opened. <br />
 under #pods for "project-name" paste:
 ```
 pod 'Stripe'
 pod 'Alamofire'
 ```
 Also be sure to uncomment the line at the top specifying the iOS. 
 ```
 platform :ios, '9.0'
 ```

 back to terminal:
 ```
 cd stripe-integration
 cd stripe-test
 pod install
 open stripe-test.xcworkspace
 ```
 Above make sure you are working in the stripe-test.xcworkspace. There is an important difference between this and your previous but still available stripe-text.xcodeproj.

 this will prep you for the next part
 
 # Integrating The Stripe SDK in Xcode
 First we need a bridging header because stripe is written in Obj-C: <br />
 Click Main.storyboard (just to make sure we are in the correct folder) <br />
 File -> new -> File... -> Header File <br />
 name the file "bridging-header" and save it <br />
 navigate to Build Settings and search Bridging Header <br />
 in the text field for "Objective-C Bridging Header" type "bridging-header.h" <br />
 go to the bridging-header.h file  <br />
 after #define and before #endif paste:
 ```
 #import <Stripe/Stripe.h>
 ```
 now go to appDelegate.swift and change the application didFinishLaunchingWithOptions function to include the following line: <br />
 (make sure you replace the test publishable key with your own)
 ```
 Stripe.setDefaultPublishableKey("pk_test_publishableKey")
 ```

 make sure to add at the top of your appDelegate.swift:
 ```
 import Stripe
 ```
 An error might continue to show up. Make sure you clean the product. The error will eventually disappear. 
 
 # Adding UI Elements
 go to Main.storyboard and add a button and change its label to "Pay". <br />
 open up a dual view with the storyboard on one side and ViewController.swift on the other side <br />
 connect the pay button to an outlet in ViewController.swift name the outlet payButtonOutlet <br />
 connect the pay button to an action in ViewController.swift name the action payButtonAction <br />
 In order to Utilize the Stripe SDK in this class you must import Stripe and Alamofire
 ```
 import Stripe
 import Alamofire
 ```
 change the class header to include STPPaymentCardTextFieldDelegate: <br />
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
 Now add the function paymentCardTextFieldDidChange to conform to the STPPaymentCardTextFieldDelegate protocol: <br />
 (also add the if statement inside of it)
 ```
 func paymentCardTextFieldDidChange(_ textField: STPPaymentCardTextField) { 
  if textField.valid {
      payButtonOutlet.isHidden = false;
   }
}
 ```
 Now make a function to make the charge on our heroku server: <br />
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
 Above you might get warnings in your print statements. This isn't a big deal, but you can go ahead and unwrap those by adding an ! at the end. (See Xcode suggestion)
 <br />

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

Now you should still see a bunch of errors and warnings here. It is because you have not imported Stripe and Alamofire yet. Be sure to do that at the top of your ViewController:
```
import Stripe
import Alamofire
```


# Getting the charge.php link
go to terminal:
```
cd 
cd stripe-integration
cd stripe-server
heroku open
```
copy the link in the address bar <br />
add to that link: /charge.php <br />
it should look something like this: https://<...>.herokuapp.com/charge.php <br />



Alright, that is it! Go ahead and run your project in Xcode. Enter in a valid or test credit card number, press pay, and then you should see a success message and a new payment appearing in your Stripe. 
 
