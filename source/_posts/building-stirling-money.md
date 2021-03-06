title: Building Stirling Money
date: 2012/5/24 15:38:52
alias: blog/473/
tags:
- Software
- Development
- C#
- Mobile
- Windows Phone
photos:
- /journal_images/Windows-Live-Writer/Building-Stirling-Money_F0B7/ScreenshotTransactions_7739df73-6a3f-4d03-95ff-84fa1544ce77.png|ScreenshotTransactions
---
Being my first fully native Windows Phone 7 application, [Stirling Money](/StirlingMoney/WindowsPhone) provided me lots of opportunities to learn more about the platform. I had written a check register application for Windows Mobile 6 and wanted to take the opportunity to update it to the new Windows Phone platform. Windows Phone 7 is a radical departure from Windows Mobile from both the user’s perspective and the developer’s perspective. Building applications for the older Windows Mobile platform utilized the Microsoft .NET Compact Framework, SQL Server Compact, and a more traditional, battleship gray, Windows Forms user interface (UI). In contrast, Windows Phone 7 uses Silverlight and XAML for the UI and LINQ to SQL to access a local database.

#### Getting Started

To get started in Windows Phone 7, the first thing I had to do is force myself to forget everything I had learned over almost 12 years of developing mobile applications for the Microsoft Pocket PC/Windows Mobile platform. Windows Phone 7 is such a radical departure that everything I had learned about building UIs and the underlying application architecture went completely out the window. Anyone who has taken the time to become an expert in a given field can tell you just how hard that can be, and it was no different for me. I found myself having to really consciously force myself out of my old Windows Mobile development patterns, but once I got to that point, I was actually very pleasantly surprised at how easy it was to build an attractive interface that performed really well and ran rings around anything I had ever done on the older Windows Mobile platform.

The first big thing that I discovered is that there three invaluable, free toolkits that really ease the development of a Windows Phone application:

*   Microsoft Silverlight Toolkit at [http://silverlight.codeplex.com](http://silverlight.codeplex.com)
*   Coding4Fun Toolkit for Windows Phone at [http://coding4fun.codeplex.com](http://coding4fun.codeplex.com)
*   Microsoft Live SDK (primarily for SkyDrive) at [http://dev.live.com](http://dev.live.com)

The Microsoft Silverlight Toolkit provides a whole host of native Silverlight controls for Windows Phone that you can drag and drop into the designer and use the Visual Studio properties window to manipulate. The Coding4Fun Toolkit provides more controls that really should have been built into the main framework if you ask me such as AboutPrompt, ProgressOverlay, RoundButton, and many more.

#### Building the UI

Even with all of these controls, building a UI in XAML is a lot like learning a foreign language. It’s just enough like XML and HTML that I’m familiar with, but with its own twists to throw me off. Data binding in XAML is particularly different than in ASP.NET or the older Windows Forms, so that was probably the biggest difference to get used to. The concept of using a view model in order to make all of the properties available to the UI was familiar to me because I use it all the time in my ASP.NET MVC development, but the binding syntax in XAML and exactly which NotifyPropertyChanged event the binding ends up listening to took some figuring out. I ended up with a number of properties in my view models that looked like the code below, with multiple NotifyPropertyChanged events, in order to get all of the related properties on the page to update when I needed them to.

```cs
private ObservableCollection<AccountViewModel> _accounts;
public ObservableCollection<AccountViewModel> Accounts
{
    get { return _accounts; }
    set
    {
        if (_accounts != value)
        {
            _accounts = value;
            if (_accounts != null)
            {
                _accounts.CollectionChanged += delegate
                {
                    NotifyPropertyChanged("Accounts");
                    NotifyPropertyChanged("TotalBalance");
                    NotifyPropertyChanged("NoDataVisibility");
                };
            }
            NotifyPropertyChanged("Accounts");
            NotifyPropertyChanged("TotalBalance");
            NotifyPropertyChanged("NoDataVisibility");
        }
    }
}
```

The other thing that I learned in the process is that the XAML editor in Visual Studio 2010, while much improved over previous editions, has some quirks when using static resources inside the `phone:PhoneApplicationPage.Resources` tag. Editing data templates in that tag doesn’t give the WYSIWYG display that editing a template inside a control does, so I often found myself editing ListItem templates inline so I could see the results in the designer and then moving the template to the Resources section once I had the design solidified.  I was pretty amazed at the amount of markup required for seemingly simple tasks. For example, one single row in the list view above required the following markup:

```xml
<DataTemplate x:Key="transactionsItemTemplate">
        <StackPanel>
            <toolkit:ContextMenuService.ContextMenu>
                <toolkit:ContextMenu>
                    <toolkit:MenuItem x:Name="menuItemEdit" Header="edit" Tag="{Binding TransactionId}" Click="menuItemEdit_Click"/>
                    <toolkit:MenuItem x:Name="menuItemDelete" Header="delete" Tag="{Binding TransactionId}" Click="menuItemDelete_Click" />
                </toolkit:ContextMenu>
            </toolkit:ContextMenuService.ContextMenu>
            <Grid>
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                </Grid.RowDefinitions>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="5"></ColumnDefinition>
                    <ColumnDefinition Width="Auto"></ColumnDefinition>
                    <ColumnDefinition Width="Auto"></ColumnDefinition>
                    <ColumnDefinition Width="*"></ColumnDefinition>
                    <ColumnDefinition Width="100"></ColumnDefinition>
                </Grid.ColumnDefinitions>
                <Rectangle Grid.Column="0" 
                           Grid.Row="0" 
                           Grid.RowSpan="2" 
                           Margin="0,2,0,2"
                           Fill="{Binding TransactionColor}"/>
                <CheckBox IsChecked="{Binding Posted, Mode=TwoWay}" 
                           Tag="{Binding TransactionId, Mode=TwoWay}"
                           Grid.Row="0"
                           Grid.Column="1"
                           Grid.RowSpan="2"
                           VerticalAlignment="Center"
                           HorizontalAlignment="Center"/>
                <TextBlock Text="{Binding CheckNumber}" 
                           FontSize="{StaticResource PhoneFontSizeNormal}"
                           Padding="0,0,10,0"
                           Grid.Row="1"
                           Grid.Column="2"
                           VerticalAlignment="Center"
                           HorizontalAlignment="Left"
                           Visibility="{Binding CheckNumberVisibility}"/>
                <TextBlock Text="{Binding Location}" 
                           FontSize="{StaticResource PhoneFontSizeMediumLarge}"
                           Margin="0,0,0,0"
                           Grid.Row="0"
                           Grid.Column="2"
                           Grid.ColumnSpan="2"
                           VerticalAlignment="Center"
                           HorizontalAlignment="Left"/>
                <TextBlock Text="{Binding Category}" 
                           FontSize="{StaticResource PhoneFontSizeNormal}"
                           Margin="0,0,0,0"
                           Grid.Row="1"
                           Grid.Column="3"
                           VerticalAlignment="Center"
                           HorizontalAlignment="Left"/>
                <TextBlock Text="{Binding Amount, StringFormat=\{0:C\}}"
                           FontSize="{StaticResource PhoneFontSizeMedium}"
                           Grid.Row="0"
                           Grid.Column="4"
                           Grid.RowSpan="2"
                           Margin="5,0,5,0"
                           TextAlignment="Right"
                           VerticalAlignment="Center"/>
        </Grid>
    </StackPanel>
</DataTemplate>
```

#### Building and Accessing Data

Database support on the phone is provided through LINQ to SQL on top of SQL Server Compact 3.5.  Having used the Microsoft Entity Framework extensively in some of my recent work at [EffectiveUI](http://www.effectiveui.com), using LINQ to SQL was relatively easy to pick up, though it does have some differences from the Entity Framework syntax that got me a bit in the early going. After that, it was simply a matter of translating from the database model to the view models and everything else just worked, which was way easier than writing the ADO.NET code and scripting the database creation that was required in Windows Mobile.

![ScreenshotBackup](/journal_images/Windows-Live-Writer/Building-Stirling-Money_F0B7/ScreenshotBackup_ac01e889-6037-458c-8da3-3fc46c009d89.png "ScreenshotBackup")

The other interesting complication was that Windows Phone does not sync files directly with the desktop. The only solution to allow the user to backup and/or restore their data from the phone is to sync with a cloud service such as [SkyDrive](http://skydrive.live.com). Fortunately the Microsoft Live SDK makes this very easy.  The following markup is all of the XAML needed to add the Sign in/Sign out button to the page:

```xml
<my:SignInButton ClientId="00000000400C1EB8" 
    Scopes="wl.signin wl.basic wl.offline_access wl.skydrive_update" 
    Branding="Skydrive" 
    Name="buttonSkyDriveSignIn" 
    SessionChanged="buttonSkyDriveSignIn_SessionChanged" />
```

After that, the C# code to handle the SkyDrive session changes was also very simple. All I had to do is listen for the SessionChanged event and check to see if the session status that was reported was Connected as shown in the snippet below.

```cs
private void buttonSkyDriveSignIn_SessionChanged(object sender, Microsoft.Live.Controls.LiveConnectSessionChangedEventArgs e)
{
    if (e.Status == LiveConnectSessionStatus.Connected)
    {
        _session = e.Session;
        _model.IsSignedIn = true;
        _client = new LiveConnectClient(_session);
        _client.GetCompleted += new EventHandler<LiveOperationCompletedEventArgs>(_client_GetCompleted);
        _client.PostCompleted += new EventHandler<LiveOperationCompletedEventArgs>(_client_PostCompleted);
        _client.UploadCompleted += new EventHandler<LiveOperationCompletedEventArgs>(_client_UploadCompleted);
        _client.DownloadCompleted += new EventHandler<LiveDownloadCompletedEventArgs>(_client_DownloadCompleted);
        _progress.Text = "Loading...";
        _progress.IsVisible = true;
        _client.GetAsync("me/skydrive/files?filter=folders", "me/skydrive/files?filter=folders");                
        _model.InfoBoxText = "Signed in.";
    }
    else
     {
         _client = null;
         _model.IsSignedIn = false;
         _model.InfoBoxText = "Not signed in.";
     }
}
```

#### Building a Trial Mode Application

Another place where the Windows Phone tools really shined is in building a trial feature into the application. Unlike in the past, where a developer had to deal with all of the registration algorithms themselves, Microsoft does a great job of providing these checks for the developer along with the hooks into the Windows Phone Marketplace to convert from trial to paid. The code to check to see if the application is running on a trial license is very simple as shown below:

```cs
private void DetermineIsTrial()
{
#if TRIAL    
    // return true if debugging with trial enabled (DebugTrial configuration is active)    
    App.IsTrial = true;
#else
    var license = new Microsoft.Phone.Marketplace.LicenseInformation();
    App.IsTrial = license.IsTrial();
#endif
}
```

This method is called when the application is loaded or is activated after the user switched to a different task and set in a static variable in the main app class so that I can check for trial functionality anywhere where that check is needed.

To convert the user to a paid user from the “Buy Now” button, I only have to execute the following code to launch the Marketplace app directly to my application page. If they purchase the application while there, the DetermineIsTrial method will be called when my application is reactivated and the user will automatically see the full functionality.

```cs
private void Button_Click(object sender, RoutedEventArgs e)
{
    MarketplaceDetailTask marketplaceDetailTask = new MarketplaceDetailTask();
    marketplaceDetailTask.Show();
}
```

#### Conclusion

Being my first fully native application for Windows Phone 7, I was really pleasantly surprised as to how easy it was to build an attractive, performant app that took advantage of local database storage and cloud backup. [Stirling Money](/StirlingMoney/WindowsPhone) took approximately 16 hours of total development time for the first release into the Windows Phone Marketplace. My next project is building a side by side comparison of building a fully-native Windows Phone application and building the same application functionality with [PhoneGap](http://www.phonegap.com)/[Apache Cordova](http://incubator.apache.org/cordova/) and doing a comparison of the two approaches in addition to the approach of building a mobile web application within a native shell that I discussed in my article, [Develop Hybrid Native and Mobile Web Apps](http://msdn.microsoft.com/magazine/hh852592), in the March 2012 issue of [MSDN Magazine](http://msdn.microsoft.com/magazine/).

As of this writing, [Stirling Money](/StirlingMoney/WindowsPhone) has been certified and published to the [Windows Phone Marketplace](http://www.windowsphone.com), although it is not currently searchable yet. When it becomes available, it will be at this [link](http://windowsphone.com/s?appid=c81aa8df-5dd6-4afe-b0eb-58a824581535). Prior to that, there is more information on my [Windows Phone Software page](/WindowsPhone). I will be publishing an additional post when the application goes live in the Marketplace.