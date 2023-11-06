# ckanext-cloudstorage

Implements support for using S3 on Google Cloud Platform

# Setup

After installing `ckanext-cloudstorage`, add it to your list of plugins in
your `.ini`:

    ckan.plugins = stats cloudstorage

If you haven't already, setup [CKAN file storage][ckanstorage] or the file
upload button will not appear.

Every driver takes two options, regardless of which one you use. Both
the name of the driver and the name of the container/bucket are
case-sensitive:

    ckanext.cloudstorage.driver = GOOGLE_STORAGE
    ckanext.cloudstorage.container_name = demo

You can find a list of driver names [here][storage] (see the `Provider
Constant` column.)

Each driver takes its own setup options. See the [libcloud][] documentation.
These options are passed in using `driver_options`, which is a Python dict.
For most drivers, this is all you need:

    ckanext.cloudstorage.driver_options = {"key": "<your public key>", "secret": "<your secret key>"}

# Support

This branch supports only Google Cloud Platform

| Provider | Uploads | Downloads | Secure URLs (private resources) |
| --- | --- | --- | --- |
| Azure    | NO | NO | NO |
| AWS S3   | NO | NO | NO |
| GCP | YES | YES | YES |

# What are "Secure URLs"?

"Secure URLs" are a method of preventing access to private resources. By
default, anyone that figures out the URL to your resource on your storage
provider can download it. Secure URLs allow you to disable public access and
instead let ckanext-cloudstorage generate temporary, one-use URLs to download
the resource. This means that the normal CKAN-provided access restrictions can
apply to resources with no further effort on your part, but still get all the
benefits of your CDN/blob storage.

    ckanext.cloudstorage.use_secure_urls = 1

This option also enables multipart uploads, but you need to create database tables
first. Run next command from extension folder:

    ckan db upgrade -p cloudstorage

With that feature you can use `cloudstorage_clean_multipart` action, which is available
only for sysadmins. After executing, all unfinished multipart uploads, older than 7 days,
will be aborted. You can configure this lifetime, example:

     ckanext.cloudstorage.max_multipart_lifetime  = 7

One-time URLs generated by CKAN are expired in an hour. This behaviour can be
changed by setting expected lifetime(in seconds) as `ckanext.cloudstorage.secure_ttl` option:

    # make one-time links valid only for 1 minute
    ckanext.cloudstorage.secure_ttl = 60

# Migrating From FileStorage

If you already have resources that have been uploaded and saved using CKAN's
built-in FileStorage, cloudstorage provides an easy migration command.
Simply setup cloudstorage as explained above, enable the plugin, and run the
migrate commands. First the following command will check if any resource that exists in the resouce tables is missing in FileStore

    ckan -c ../ckan/development.ini cloudstorage check-resources

The following command will migrate the resources:

    ckan -c ../ckan/development.ini cloudstorage migrate

The following command will migrate all the images and the assets:

    ckan -c ../ckan/development.ini cloudstorage assets-to-gcp


# Notes

1. You should disable public listing on the cloud service provider you're
   using, if supported.
2. On this branch things like group and organization images are supported and they also use the S3 storage.

# FAQ

- *DataViews aren't showing my data!* - did you setup CORS rules properly on
  your hosting service? ckanext-cloudstorage can try to fix them for you automatically,
  run:

        ckan -c=<CKAN config> cloudstorage fix-cors <list of your domains> 


[libcloud]: https://libcloud.apache.org/
[ckan]: http://ckan.org/
[storage]: https://libcloud.readthedocs.io/en/latest/storage/supported_providers.html
[ckanstorage]: http://docs.ckan.org/en/latest/maintaining/filestore.html#setup-file-uploads
