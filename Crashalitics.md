#### Once you have the dsym in the project and have configured firebase add a script

* Click the Build Phases tab, then complete the following steps so that Xcode can process your dSYMs and upload the files.
* Click add > New Run Script Phase.
* Make sure this new Run Script phase is your project's last build phase; otherwise, Crashlytics can't properly process dSYMs.
* Expand the new Run Script section.
