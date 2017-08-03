# Xamarine_Android
Basic Android Tutorial

Launch Xamarin Studio from the Applications folder or from Spotlight. This opens the start page:

Launch Xamarin Studio
Click New Solution... to create a new project:

Create a new project
In the Choose a template for your new project dialog, click Android > App and select the Android App template. Click Next.

Choose the Android App template
In the Configure your Android app dialog, name the new app Phoneword and click Next:

Name the project Phoneword
In the Configure your new project dialog, leave the Solution and Project names set to Phoneword and click Create to create the project. Be sure to uncheck the checkbox for Xamarin Test Cloud (if the Xamarin Insights checkbox is shown, uncheck it as well):

Create project
After the new project is created, expand the Resources folder and then the layout folder in the Solution pad. Double-click Main.axml to open it in the Android Designer. This is the layout file for the screen:

Open Main.axml
Select the Hello World, Click Me! Button on the design surface and press the Delete key to remove it. From the Toolbox (the area on the right), enter text into the search field and drag a Text (Large) widget onto the design surface (the area in the center):

Add large text widget
With the Text (Large) widget selected on the design surface, you can use the Properties pad to change the Text property of the Text (Large) widget to Enter a Phoneword: as shown below:

Set large text widget properties
Next, drag a Plain Text widget from the Toolbox to the design surface and place it underneath the Text (Large) widget. Notice that you can use the search field to help locate widgets by name:

Add plain text widget
With the Plain Text widget selected on the design surface, you can use the Properties pad to change the Id property of the Plain Text widget to @+id/PhoneNumberText and change the Text property to 1-855-XAMARIN:

Set plain text widget properties
Drag a Button from the Toolbox to the design surface and place it underneath the Plain Text widget:

Add a button
With the Button selected on the design surface, you can use the Properties pad to change the Id property of the Button to @+id/TranslateButton and change the Text property to Translate:

Configure as the translate button
Next, drag a second Button from the Toolbox to the design surface and place it underneath the Translate button:

Add a second button
With the Button selected on the design surface, you can use the Properties pad to change the Id property of the Button to @+id/CallButton and change the Text property to Call:

Configure as the call button

Save your work by pressing ⌘ + S.
Now, add some code to translate phone numbers from alphanumeric to numeric. Add a new file to the project by clicking the gear icon next to the Phoneword project in the Solution pad and choosing Add > New File...:

Add a new file to the project
In the New File dialog, select General > Empty Class, name the new file PhoneTranslator, and click New:

The new file is named PhoneTranslator.cs
This creates a new empty C# class for us. Remove all of the template code and replace it with the following code:

using System.Text;
using System;
namespace Core
{
    public static class PhonewordTranslator
    {
        public static string ToNumber(string raw)
        {
            if (string.IsNullOrWhiteSpace(raw))
                return "";
            else
                raw = raw.ToUpperInvariant();

            var newNumber = new StringBuilder();
            foreach (var c in raw)
            {
                if (" -0123456789".Contains(c))
                    newNumber.Append(c);
                else {
                    var result = TranslateToNumber(c);
                    if (result != null)
                        newNumber.Append(result);
                }
                // otherwise we've skipped a non-numeric char
            }
            return newNumber.ToString();
        }
        static bool Contains (this string keyString, char c)
        {
            return keyString.IndexOf(c) >= 0;
        }
        static int? TranslateToNumber(char c)
        {
            if ("ABC".Contains(c))
                return 2;
            else if ("DEF".Contains(c))
                return 3;
            else if ("GHI".Contains(c))
                return 4;
            else if ("JKL".Contains(c))
                return 5;
            else if ("MNO".Contains(c))
                return 6;
            else if ("PQRS".Contains(c))
                return 7;
            else if ("TUV".Contains(c))
                return 8;
            else if ("WXYZ".Contains(c))
                return 9;
            return null;
        }
    }
}
Save the changes to the PhoneTranslator.cs file by choosing File > Save (or by pressing ⌘ + S), then close the file. Ensure that there are no compile-time errors by rebuilding the solution.
The next step is to add code to wire up the user interface by adding the backing code into the MainActivity class. Double-click MainActivity.cs in the Solution Pad to open it:

Open MainActivity.cs
Begin by wiring up the Translate button. In the MainActivity class, find the OnCreate method. Add the button code inside OnCreate, below the base.OnCreate(bundle) and SetContentView (Resource.Layout.Main) calls. Remove the template button handling code so that the OnCreate method resembles the following:

using System;
using Android.App;
using Android.Content;
using Android.Runtime;
using Android.Views;
using Android.Widget;
using Android.OS;

namespace Phoneword
{
    [Activity (Label = "Phoneword", MainLauncher = true)]
    public class MainActivity : Activity
    {
        protected override void OnCreate (Bundle bundle)
        {
            base.OnCreate (bundle);

            // Set our view from the "main" layout resource
            SetContentView (Resource.Layout.Main);

            // Our code will go here
        }
    }
}
Next, a reference is needed to the controls that were created in the layout file with the Android Designer. Add the following code inside the OnCreate method (after the call to SetContentView):

// Get our UI controls from the loaded layout:
EditText phoneNumberText = FindViewById<EditText>(Resource.Id.PhoneNumberText);
Button translateButton = FindViewById<Button>(Resource.Id.TranslateButton);
Button callButton = FindViewById<Button>(Resource.Id.CallButton);
Add code that responds to user presses of the Translate button by adding the following code to the OnCreate method (after the lines added in the last step):

// Disable the "Call" button
callButton.Enabled = false;

// Add code to translate number
string translatedNumber = string.Empty;

translateButton.Click += (object sender, EventArgs e) =>
{
    // Translate user's alphanumeric phone number to numeric
    translatedNumber = Core.PhonewordTranslator.ToNumber(phoneNumberText.Text);
    if (String.IsNullOrWhiteSpace(translatedNumber))
    {
        callButton.Text = "Call";
        callButton.Enabled = false;
    }
    else
    {
        callButton.Text = "Call " + translatedNumber;
        callButton.Enabled = true;
    }
};
Add code that responds to user presses of the Call button. Place the following code below the code for the Translate button:

callButton.Click += (object sender, EventArgs e) =>
{
    // On "Call" button click, try to dial phone number.
    var callDialog = new AlertDialog.Builder(this);
    callDialog.SetMessage("Call " + translatedNumber + "?");
    callDialog.SetNeutralButton("Call", delegate {
           // Create intent to dial phone
           var callIntent = new Intent(Intent.ActionCall);
           callIntent.SetData(Android.Net.Uri.Parse("tel:" + translatedNumber));
           StartActivity(callIntent);
       });
    callDialog.SetNegativeButton("Cancel", delegate { });

    // Show the alert dialog to the user and wait for response.
    callDialog.Show();
};
Finally, give the application permission to place a phone call. Open the project options by right-clicking Phoneword in the Solution pad and selecting Options:

Open application permissions

In the Project Options dialog, select Build > Android Application. In the Required Permissions section, enable the CallPhone permission:

Enable the CallPhone permission
Save your work and build the application by selecting Build > Build All (or by pressing ⌘ + B). If the application compiles, you will get a success message at the top of Xamarin Studio:

Successful build

If there are errors, go through the previous steps and correct any mistakes until the application builds successfully. If you get a build error such as, Resource does not exist in the current context, verify that the namespace name in MainActivity.cs matches the project name (Phoneword) and then completely rebuild the solution. If you still get build errors, verify that you have installed the latest Xamarin.Android and Xamarin Studio updates.
Now that you have a working application, it's time to add the finishing touches! Start by editing the Label for MainActivity. The Label is what Android displays at the top of the screen to let users know where they are in the application. At the top of the MainActivity class, change the Label to Phone Word as shown here:

namespace Phoneword
{
    [Activity (Label = "Phone Word", MainLauncher = true)]
    public class MainActivity : Activity
    {
        ...
    }
}
Next, set the application icon. Open the downloaded and unzipped Xamarin App Icons set. Expand the drawable-hdpi folder under Resources and remove the existing Icon.png by right-clicking it and selecting Remove:

Remove existing icon

When the following dialog box is displayed, select Delete:

Delete icon
Next, right-click the drawable-hdpi folder and select Add > Add Files:

Add files
From the selection dialog, navigate to the unzipped Xamarin App Icons directory and open the drawable-hdpi folder. Select Icon.png:

Select icon file
In the Add File to Folder dialog box, select Copy the file into the directory and click OK:

Copy the file to the directory
Repeat these steps for each of the drawable- folders until the contents of the drawable- Xamarin App Icons folders are copied to their counterpart drawable- folders in the Phoneword project:

Add drawable folders to Resources

These folders provide different resolutions of the icon so that it renders correctly on different devices with different screen densities.
Finally, test the application by deploying it to an Android emulator. In Xamarin Studio, select a virtual device (under Virtual Devices) and click the play button in the upper left corner:

Select virtual device

As shown in this screenshot, the Nexus 4 (KitKat) (API 19) virtual device was selected.
After Xamarin Studio loads the application into the virtual device, the Phoneword app is automatically started. The screenshots below illustrate the Phoneword application running in an Android SDK emulator configured as a Nexus 5 running Lollipop. Clicking the Translate button updates the text of the Call button, and clicking the Call button causes the call dialog to appear as shown on the right:

Phoneword running in emulator
Congratulations on completing your first Xamarin.Android application! Now it's time to dissect the tools and skills you have just learned.
