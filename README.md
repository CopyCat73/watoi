# Whatsapp Android To iOS Importer

Migrate Whatsapp chats history from Android to iOS.

## Caveats

* Media files and shared locations are not imported (got placeholders instead)
* Messages from contacts that changed ids (phone numbers) are not linked

## Prerequisites

* Mac with installed Xcode and iTunes
* Decrypted `msgstore.db` from Android:
  * Download and unzip https://github.com/p4r4d0x86/WhatsApp-Key-DB-Extractor/archive/v4.7-E1.0.zip
  * Download http://whatcrypt.com/WhatsApp-2.11.431.apk
  * Put the downloaded apk in the tmp folder of the unzipped github extractor project. Delete LegacyWhatsApp.apk and rename the WhatsApp apk to LegacyWhatsApp.apk
  * Follow the steps from here:https://forum.xda-developers.com/showthread.php?p=81902381#post81902381 (you have already done steps 3 to 6 of the first part). 

* Installed and activated Whatsapp on your iDevice
* `Whatsapp.ipa` of the same version (google will help)

## Step-by-step guide

* Check that Whatsapp is activated on iDevice. You should see the list of *group* chats
  when you open the app. Most likely, there won't be any messages prior to moving to iOS.
  You can even send/receive a message or two to be sure that there is something to back up.
* Build the migration utility (I'll assume `~/Downloads` folder):

      cd ~/Downloads
      git clone https://github.com/residentsummer/watoi
      cd watoi
      xcodebuild -project watoi.xcodeproj -target watoi

* Create an unencrypted backup to local computer (not iCloud) with iTunes.
  Find the latest backup in `~/Library/Application Support/MobileSync/Backup`.
* Locate Whatsapp database file inside the backup and copy it somewhere:

      $ sqlite3 <backup>/Manifest.db "select fileID from Files where relativePath = 'ChatStorage.sqlite' and domain like '%whatsapp%';"
      abcdef01234567890
      $ cp <backup>/ab/abcdef01234567890 ~/Downloads/watoi/ChatStorage.sqlite

> Explanation: the alphanumeric ID (`abcdef01234567890` in the example above) you'll get after running the first command is the name
 of the 'blob', that contains Whatsapp database. Location of this file inside the backup depends on the version of iTunes used - it's
 either `<backup path>/<first two characters of the blob ID>/<blob ID>` or just `<backup path>/<blob ID>`. In our example this will  be
 `<backup path>/ab/abcdef01234567890` or `<backup path>/abcdef01234567890`.

* Extract the contents of `Whatsapp.ipa` (we'll need CoreData description files):

      cd ~/Downloads/watoi
      unzip ~/Downloads/WhatsApp_Messenger_x.y.z.ipa -d app

* Backup original database and run the migration:

      cp ChatStorage.sqlite ~/Documents/SafePlace/
      build/Release/watoi <path-to-msgstore.db> ChatStorage.sqlite app/Payload/WhatsApp.app/Frameworks/Core.framework/WhatsAppChat.momd

* Replace database file inside the backup with the updated one:

      cp ChatStorage.sqlite "~/Library/Application Support/MobileSync/Backup/<backup>/ab/abcdef01234567890"

* Restore the backup with iTunes

## Troubleshooting

[![Gitter](https://badges.gitter.im/gitterHQ/gitter.svg)](https://gitter.im/residentsummer_watoi/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

## More TODOs

* Automate backup editing (skip manual ChatStorage locate/extract/replace)
* Better command-line parsing (better than none)
