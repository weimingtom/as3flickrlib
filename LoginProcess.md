# Introduction #

Some methods require a user to be authenticated before being invoked.  For example, calling the delete photo method will fail unless the user gives your application permission to delete her photos.

The following sample code snippet demonstrate the login process

# Flex Based Authentication #

The library contains a Flex control that provides a UI and code for authenticating an application with Flickr. You can find a quick overview of the control at:
http://www.mikechambers.com/blog/2008/08/12/flex-based-flickr-api-authorization-control/


# Sample code for authenticating a user #
```
			/**
			 * Some methods require authentication, so this is how you
			 * would start the login sequence.  We need to set the secret
			 * assigned to our particular application, and then get
			 * a frob used for authentication.
			 */
			private function startLoginSequence():void {
				service.addEventListener( FlickrResultEvent.AUTH_GET_FROB, getFrobResponse );
				service.auth.getFrob();
			}
			
			/**
			 * When we receive the frob, we need to construct a login link
			 * and open a browser window for the user to log into the flickr
			 * site.  Use the service getLoginURL method to construct a 
			 * login link with the frob we received, and pass along the
			 * permission we'd like to be granted from the user.
			 */
			private function getFrobResponse( event:FlickrResultEvent ):void {
				debug.text += "getFrobResponse: success = " + event.success + "\n";
				
				if ( event.success ) {
					// Have the service construct a login url for us with the
					// authentication frob, and request the user that we'd like
					// to have DELETE access to their data
					frob = String( event.data.frob );
					var auth_url:String = service.getLoginURL( frob, AuthPerm.DELETE );
					
					// Open a new browser window to authenticate the user
					// and grant our application permission
					navigateToURL( new URLRequest( auth_url ), "_blank" );
										
					// Show the alert saying  they need to authenticate on the 
					// flickr site.  when the alert closes, we need to get the 
					// token then to get their logged-in status
					Alert.show( "This application requires that you authenticate"
								+ " on Flickr.com before proceeding.  Please log in"
								+ " to Flickr in the separate browser window that"
								+ " opened.  After you have successfully logged in,"
								+ " press 'OK' below to continue",
								"Authentication Required",
								Alert.OK | Alert.CANCEL,
								null,
								onCloseAuthWindow );
								
				} else {
					debug.text += "error code: " + event.data.error.errorCode + "\n";
					debug.text += "error message: " + event.data.error.errorMessage + "\n";
				}
			}
			
			/**
			 * After the alert closes, if they pressed the OK button we
			 * assume that they logged into Flickr, so try to get their
			 * auth token that we can use throughout the rest of our app
			 */
			private function onCloseAuthWindow( event:* ):void {
				// Only process if they pressed OK
				if ( event.detail == Alert.OK ) {
					// Get their authentication token, and call getTokenResponse
					// when it's available
					service.addEventListener( FlickrResultEvent.AUTH_GET_TOKEN, getTokenResponse );
					service.auth.getToken( frob );	
				}
			}
			
			/**
			 * This completes the login process.  When the user is successfully
			 * authenticated and the application has permission to use their
			 * data, there will be a token that flickr assigns to us.
			 */
			private function getTokenResponse( event:FlickrResultEvent ):void {
				debug.text += "getTokenResponse: success = " + event.success + "\n";
				
				if ( event.success ) {
					var authResult:AuthResult = AuthResult( event.data );
					// dump the object internals for debugging
					debug.text += ObjectUtil.toString( authResult ) + "\n";
					
					// Assign the token and permission to the service so that
					// all calls that require authentication have their values
					// populated
					service.token = authResult.token;
					service.permission = authResult.perms;
					
					// Save the token in a shared object so that when the application
					// loads again we can re-authenticate automatically
					var flickrCookie:SharedObject = SharedObject.getLocal( "FlickrServiceTest" );
					flickrCookie.data.auth_token = service.token;
					flickrCookie.flush();
					
					// Update the UI to show the currently logged in username
					username.text = authResult.user.username + " (" + authResult.user.fullname + " )";
					permission.text = service.permission;
					
					debug.text += "token: " + service.token + "\n";
					debug.text += "perms: " + service.permission + "\n";
					
					// Toggle the login/logout buttons
					login.visible = false;
					logout.visible = true;
					
				} else {
					debug.text += "error code: " + event.data.errorCode + "\n";
					debug.text += "error message: " + event.data.errorMessage + "\n";
				}
			}
```