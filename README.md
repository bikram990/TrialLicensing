# Time-Based Trial App Licensing

Adds time-based trial and easy license verification using [CocoaFob](https://github.com/glebd/cocoafob) to your macOS app.

[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)

**For setup instructions of your own store and app preparation with FastSpring, have a look at my book [_Make Money Outside the Mac App Store_](https://christiantietze.de/books/make-money-outside-mac-app-store-fastspring/)!**

## Usage 

* Create an `AppLicensing` instance.
* Conform to `AppLicensingDelegate` in your project to receive license change notifications.
* Set up and start the trial.

Example:

    import TrialLicense

    let publicKey = [
            "-----BEGIN DSA PUBLIC KEY-----\n",
            // ...
            "-----END DSA PUBLIC KEY-----\n"
        ].join("")
    let configuration = LicenseConfiguration(appName: "AmazingApp!", publicKey: publicKey)
    
    class MyApp: AppLicensingDelegate {
        
        init() {
            
            AppLicensing.setUp(
                configuration: configuration,
                initialTrialDuration: Days(30),
                delegate: self,
                
                // Get notified about initial state to unlock the app immediately:
                fireInitialState: true)
        }
        
        func licenseDidChange(licenseInformation: LicenseInformation) {
            
            switch licenseInformation {
            case .onTrial(_):
                // Changing back to trial may be possible if you support unregistering
                // form the app (and the trial period is still good.)
                return
    
            case .registered(_):
                // For example:
                //   displayThankYouAlert()
                //   unlockApp()
    
            case .trialUp:
                // For example:
                //   displayTrialUpAlert()
                //   lockApp()
                //   showRegisterApp()
            }
        }
        
        func didEnterInvalidLicenseCode(name: String, licenseCode: String) {
            
            // For example:
            //   displayInvalidLicenseAlert()
            // -- or show an error label in the license window.
        }
    }
    
    let myApp = MyApp()

## Components

`LicenseInformation` reveals the state your app is in:

    enum LicenseInformation {
        case registered(License)
        case onTrial(TrialPeriod)
        case trialUp
    }

The associated types provide additional information, for example to display details in a settings window or show remaining trial days in the title bar of your app.

`License` represents a valid name--license code pair:

    struct License {
        let name: String
        let licenseCode: String
    }

`TrialPeriod` encapsulates the duration of the trial.

    struct TrialPeriod {
        let startDate: Date
        let endDate: Date
    }

`TrialPeriod` also provides these convenience methods:

* `ended() -> Bool`
* `daysLeft() -> Days`

... where `Days` encapsulates the remainder for easy conversion to `TimeInterval` and exposing `userFacingAmount: Int` for display.

## License

Copyright (c) 2016 by [Christian Tietze](http://christiantietze.de/).

The MIT License. See the LICENSE file for details.