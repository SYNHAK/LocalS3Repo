LocalS3Repo
===========

This extension implements the Amazon S3 as a filerepo for MediaWiki.

This software is originally at [MediaWiki Extension:LocalS3Repo](http://www.mediawiki.org/wiki/Extension:LocalS3Repo) and released under GPL. Documentation copied from the website with minor edits.

## What can this extension do?

This extension implements the Amazon S3 as a filerepo. The local model was used as a base, and the wiki will use the S3 instead of the local disk drive for the storage of all uploaded files.

## Usage

Once installed, it will operate automatically. There are no user interface issues. You have to have an account with Amazon S3, explained here.

## Download instructions

Please download LocalS3Repo.zip and unzip it to $IP/extensions/, with folders. The files should be in the $IP/extensions/LocalS3Repo/ directory. Note: $IP stands for the root directory of your MediaWiki installation, the same directory that holds LocalSettings.php.

## Installation

To install this extension, add the following to LocalSettings.php:

```php
// s3 filesystem repo
$wgUploadDirectory = 'wiki-images';
$wgUploadS3Bucket = '---Your S3 bucket---';
$wgUploadS3SSL = false; // true if SSL should be used
$wgPublicS3 = true; // true if public, false if authentication should be used
$wgS3BaseUrl = "http".($wgUploadS3SSL?"s":"")."://s3.amazonaws.com/$wgUploadS3Bucket";
$wgUploadBaseUrl = "$wgS3BaseUrl/$wgUploadDirectory";
$wgLocalFileRepo = array(
        'class' => 'LocalS3Repo',
        'name' => 's3',
        'directory' => $wgUploadDirectory,
        'url' => $wgUploadBaseUrl ? $wgUploadBaseUrl . $wgUploadPath : $wgUploadPath,
        'urlbase' => $wgS3BaseUrl ? $wgS3BaseUrl : "",
        'hashLevels' => $wgHashedUploadDirectory ? 2 : 0,
        'thumbScriptUrl' => $wgThumbnailScriptPath,
        'transformVia404' => !$wgGenerateThumbnailOnParse,
        'initialCapital' => $wgCapitalLinks,
        'deletedDir' => $wgUploadDirectory.'/deleted',
        'deletedHashLevels' => $wgFileStore['deleted']['hash'],
        'AWS_ACCESS_KEY' => '---Your S3 access key---',
        'AWS_SECRET_KEY' => '---Your S3 secret key---',
        'AWS_S3_BUCKET' => $wgUploadS3Bucket,
        'AWS_S3_PUBLIC' => $wgPublicS3,
        'AWS_S3_SSL' => $wgUploadS3SSL
);
require_once("$IP/extensions/LocalS3Repo/LocalS3Repo.php");
// s3 filesystem repo - end
```

To transfer your files from an existing wiki:

* Create the bucket and the "folder" (the S3Fox tool for FireFox works well for this)
* Transfer all the files/folders in your existing wiki's image directory to your "folder".
* Set the ACL for the folder (and subfolders, etc) to be readable by everyone, if you do not use authentication

I have tested this with Windows 7 and with Linux, with Mediawiki versions 1.15.4 and 1.16.0beta3. I currently have a wiki running on Linux with this extension. I have tested the funcionality of upload, thumbnails, delete, archive, and rollback. There could be some other functions which some other extensions use which I didn't implement or test, watch out!

This extension isn't by any means perfect, because the files are sent first to the wiki server, and from there they are sent to the S3 system on uploads. On file reading, they come directly from the S3 system, which is correct. I am waiting for a new upload screen which has a progress bar, then I will implement sending the files directly to the S3 system on upload. So for now, the file size limit of your PHP server will be the file size limit. I deal with that on my system by doing it manually, because I have few files larger than the limit.

Note that the S3.php file, from Donovan Schönknecht, is included in the zip file, and requires that curl be installed on your PHP server (check your phpinfo).

### Configuration parameters
* $wgUploadS3Bucket --> The name of the bucket you created for the wiki
* $wgUploadDirectory --> The "folder" under the bucket where you wish the files to be stored
* $wgUploadS3SSL --> true if SSL should be used
* $wgPublicS3 --> true if public S3 file access, false if signature authentication should be used
* 'AWS_ACCESS_KEY' --> Your S3 access key, a long string of characters assigned by Amazon
* 'AWS_SECRET_KEY' --> -Your S3 secret key, a long string of characters assigned by Amazon

## See also
* [Manual:Integration_with_S3](http://www.mediawiki.org/wiki/Manual:Integration_with_S3)
