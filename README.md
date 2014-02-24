# Nginx Upload Module 2.2.1-vxl

This build is forked from https://github.com/vkholodkov/nginx-upload-module branch 2.2.

nginx-upload-module adds a much desired feature in Nginx to buffer file uploads to files before passing it to the upstreams. This improves efficiencies significantly as nginx is much better at buffering files than dynamic backends such as Rails.

Starting with Nginx 1.3.9, due to internal API changes in nginx, this module no longer compiles. The author of this module has moved onto other things and no longer maintains the repo. His position can be read here: https://github.com/vkholodkov/nginx-upload-module/issues/41.

This fork fixes the compilation with 1.3.9+ (including 1.5.x), and also adds support to allow PUT and PATCH methods. The original module only accepts POST requests, which makes it more difficult to use with RESTful upstream handlers.

### RESTful locations

One thing to note with RESTful locations is you do NOT want to enable upstream for all types of requests for that location. nginx-upload-module will reject any requests that's not POST/PUT/PATCH.

Example:

```
if ( $request_method ~ ^(POST|PUT|PATCH)$ ) {
    # Pass altered request body to this location
    upload_pass   @test;

    # Store files to this directory
    # The directory is hashed, subdirectories 0 1 2 3 4 5 6 7 8 9 should exist
    upload_store /tmp 1;

    # Allow uploaded files to be read only by user
    upload_store_access user:r;

    # Set specified fields in request body
    upload_set_form_field "${upload_field_name}_name" $upload_file_name;
    upload_set_form_field "${upload_field_name}_content_type" $upload_content_type;
    upload_set_form_field "${upload_field_name}_path" $upload_tmp_path;

    # Inform backend about hash and size of a file
    upload_aggregate_form_field "${upload_field_name}_md5" $upload_file_md5;
    upload_aggregate_form_field "${upload_field_name}_size" $upload_file_size;

    upload_pass_form_field "^submit$|^description$";
}
```