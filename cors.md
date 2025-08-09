# CORS

If you want to allow your APIs to be consumed from external domains
you need to understand the mechanism of `Cross-Origin Resource Sharing` aka **CORS**.

<p class="tip"><a target="_blank" href="https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS">Cross-Origin Resource Sharing (CORS)
</a></p>

`Lightpack` framework comes with a filter to support CORS requests made by browsers while
accessing your APIs. 

The class `Lightpack\Filters\CorsFilter` adds sufficient headers in the **before()**
and **after()** methods to allow preflight requests made by browsers using `OPTIONS` request method.

## Configuration

If you plan to modify **CORS** related headers configurations, please run following command to create `config/cors.php` configuration file.

```cli
php console create:config --support=cors
```