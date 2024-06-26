
This project is based on Apple's original example code for SFAuthorizationPluginView. It has been modified to address specific issues encountered in previous versions, including a bug related to certificate handling in screensaver mode.

When sending an HTTP request while in screensaver mode, the intermediate and root certificates from the received certificate chain are missing.

Based on the work from https://github.com/skycocker/NameAndPassword, the following modifications were made:
- add http request when init

Follow these steps to set up the project:
1. Run the command `security authorizationdb write authenticate < authenticate.plist` in the directory.
2. Run the command `security authorizationdb write system.login.screensaver < system.login.screensaver.plist` in the directory.
3. Run the command `xattr -r -d "com.apple.quarantine" "/Library/Security/SecurityAgentPlugins/NameAndPassword.bundle"`
4. reboot

After reboot, lock then unlock the screensaver and performing the privilege escalation, execute the command
`log show --predicate 'message contains "macOS-Screensaver-Cert-Bug"' --last 1h`
to observe the loss of HTTP connection certificates during screensaver unlock


2019 update by Antoine Bellanger (http://antoinebellanger.ch): fixed nib and ARCHS

How does it work ?

A `SFAuthorizationPluginView` Introduction (https://antoinebellanger.ch/blog-post/sfauthorizationpluginview-introduction)

/*

File: NameAndPassword.m

Abstract: SFAuthorizationPluginView subclass

Version: 1.0

Disclaimer: IMPORTANT:  This Apple software is supplied to you by Apple
Computer, Inc. ("Apple") in consideration of your agreement to the
following terms, and your use, installation, modification or
redistribution of this Apple software constitutes acceptance of these
terms.  If you do not agree with these terms, please do not use,
install, modify or redistribute this Apple software.

In consideration of your agreement to abide by the following terms, and
subject to these terms, Apple grants you a personal, non-exclusive
license, under Apple's copyrights in this original Apple software (the
"Apple Software"), to use, reproduce, modify and redistribute the Apple
Software, with or without modifications, in source and/or binary forms;
provided that if you redistribute the Apple Software in its entirety and
without modifications, you must retain this notice and the following
text and disclaimers in all such redistributions of the Apple Software. 
Neither the name, trademarks, service marks or logos of Apple Computer,
Inc. may be used to endorse or promote products derived from the Apple
Software without specific prior written permission from Apple.  Except
as expressly stated in this notice, no other rights or licenses, express
or implied, are granted by Apple herein, including but not limited to
any patent rights that may be infringed by your derivative works or by
other works in which the Apple Software may be incorporated.

The Apple Software is provided by Apple on an "AS IS" basis.  APPLE
MAKES NO WARRANTIES, EXPRESS OR IMPLIED, INCLUDING WITHOUT LIMITATION
THE IMPLIED WARRANTIES OF NON-INFRINGEMENT, MERCHANTABILITY AND FITNESS
FOR A PARTICULAR PURPOSE, REGARDING THE APPLE SOFTWARE OR ITS USE AND
OPERATION ALONE OR IN COMBINATION WITH YOUR PRODUCTS.

IN NO EVENT SHALL APPLE BE LIABLE FOR ANY SPECIAL, INDIRECT, INCIDENTAL
OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) ARISING IN ANY WAY OUT OF THE USE, REPRODUCTION,
MODIFICATION AND/OR DISTRIBUTION OF THE APPLE SOFTWARE, HOWEVER CAUSED
AND WHETHER UNDER THEORY OF CONTRACT, TORT (INCLUDING NEGLIGENCE),
STRICT LIABILITY OR OTHERWISE, EVEN IF APPLE HAS BEEN ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.

Copyright © 2006 Apple Computer, Inc., All Rights Reserved

*/

NameAndPassword

This project is an example of a Authorization Plug-in that uses an SFAuthorizationPluginView to present UI to the user.  The NameAndPassword.bundle should be installed in "/System/Library/CoreServices/SecurityAgentPlugins/"


An good way to test your plugin is to create a right in the Policy Database that ask for that right using the code below:

- (void)askForRight:(NSString *)inRightName
{
	AuthorizationItem		right = { NULL, 0, NULL, 0 };
	AuthorizationRights		rightSet = { 1, &right };
	AuthorizationFlags		flags = kAuthorizationFlagDefaults | kAuthorizationFlagInteractionAllowed | kAuthorizationFlagExtendRights | kAuthorizationFlagPreAuthorize;
	OSStatus				status;
	SFAuthorization			*authorization;
	
	right.name = [inRightName cStringUsingEncoding: NSASCIIStringEncoding];
	
	authorization = [SFAuthorization authorization];
	if (!authorization)
	{
		NSLog("couldn't create authorization.");
		return;
	}
	
	status = [authorization permitWithRights:&rightSet flags:flags environment:kAuthorizationEmptyEnvironment authorizedRights:NULL];
	if (status != errAuthorizationSuccess)
	{
		NSLog("auth failed with err %d.", status);
		return;
	}
}


NOTE: The WWDC 2006 build of Mac OS X 10.5 likely won't authenticate properly.  But it will allow you to test your SFAuthorizationPluginView.
