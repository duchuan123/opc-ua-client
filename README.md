![robot][1]

# opc-ua-client
Looking for samples? See https://github.com/convertersystems/opc-ua-samples

*New!* Install package 'Workstation.UaClient' from Nuget to get the latest release for your hmi project.

*New!* Supports Universal Windows Platform (UWP) and Windows Presentation Framework (WPF) applications.

Build a free HMI using OPC Unified Architecture and Visual Studio. With this library, your app can browse, read, write and subscribe to the live data published by the OPC UA servers on your network.

Get the companion Visual Studio extension 'Workstation.UaBrowser' and you can:
- Browse OPC UA servers directly from the Visual Studio IDE.
- Drag and drop the variables, methods and events onto your view model.
- Use XAML bindings to connect your UI elements to live data.

### Main Types
- UaTcpSessionClient - A client for browsing, reading, writing and subscribing to nodes of your OPC UA server. Connects and reconnects automatically. 100% asynchronous.
- SubscriptionAttribute - An attribute for your view models. Permits UaTcpSessionClient to automatically create and delete subscriptions on the server and deliver data change and event notifications to properties.
- MonitoredItemAttribute - An attribute for properties that indicates the property will receive data change or event notifications from the server.

```
    public partial class App : Application
    {
        protected override void OnStartup(StartupEventArgs e)
        {
            // Prepare for constructing the shared session client.
            var appDescription = new ApplicationDescription()
            {
                ApplicationName = "Workstation.StatusHmi",
                ApplicationUri = $"urn:{System.Net.Dns.GetHostName()}:Workstation.StatusHmi",
                ApplicationType = ApplicationType.Client
            };
            var appCertificate = appDescription.GetCertificate();
            var endpointUrl = StatusHmi.Properties.Settings.Default.EndpointUrl;

            // Create the session client for the app.
            this.session = new UaTcpSessionClient(appDescription, appCertificate, null, endpointUrl);

            // Create the MainViewModel subscription.
            var subscription = this.session.CreateSubscription<MainViewModel>();

            // Create and show the MainView.
            var view = new MainView { DataContext = subscription };

            view.Show();
        }
    }
    
    [Subscription(publishingInterval: 500, keepAliveCount: 20)]
    public class MainViewModel : ViewModelBase
    {
        /// <summary>
        /// Gets the value of ServerStatusCurrentTime.
        /// </summary>
        [MonitoredItem(nodeId: "i=2258")]
        public DateTime ServerStatusCurrentTime
        {
            get { return this.serverStatusCurrentTime; }
            private set { this.SetProperty(ref this.serverStatusCurrentTime, value); }
        }

        private DateTime serverStatusCurrentTime;
    }
```
### Releases

v1.4.0 UaTcpSessionClient now calls a asynchronous function you provide when connecting to servers that request a UserNameIdentity. Depreciated ISubscription and replaced with SubscriptionAttribute to specify Subscription parameters.  If ViewModelBase implements ISetDataErrorInfo and INotifyDataErrorInfo then it will record any error messages that occur when creating, writing or publishing a MonitoredItem. Diagnostics now use EventSource for logging. Added Debug, Console and File EventListeners. 

v1.3.0 Depreciated Subscription base class in favor of ISubscription interface to allow freedom to choose whatever base class you wish for your view models.
   
v1.2.0 Client, Subscription and Channel class constructors have new optional arguments. Corresponding property setters are removed to prevent changes after construction. Fixed some threading issues: Subscription's publish on thread pool, viewmodel's update on dispatcher thread. 

v1.1.0 Simplified Subscription base class to automatically subscribe for data change and event notifications when constructed, re-subscribe if the server reboots, and un-subscribe when garbage collected.   

v1.0.0 First commit. Includes UaTcpSessionChannel for 'opc.tcp' servers. Supports security up to Basic256Sha256. Automatically creates self-signed X509Certificate with 2048bit key.

[1]: robot6.jpg  
