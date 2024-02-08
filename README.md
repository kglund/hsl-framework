# hsl-framework
An iRule framework to stand up application High Speed Logging on F5 Big-IP load balancers with a variety of profiles

## prerequisites

1. A vip must have either a TCP profile or an HTTP profile applied to process this framework.
2. These rules assume the existence of a HSL log publisher: https://my.f5.com/manage/s/article/K50040950
3. These rules are built with the assumption that the following string datagroups exist:
    - global_config -- general configurations, including HSL on/off switch
    - hsl_log_headers -- list of URIs OR VIPs on which header logging should be enabled
    - hsl_general_logger -- list of URIs OR VIPs on which detailed logging should be enabled
    - hsl_errlog -- list of URIs OR VIPs on which server errors should be logged
    - preferred_client_ciphers -- list of preferred ciphers for client-side handshake
    - preferred_server_ciphers -- list of preferred ciphers for server-side handshake

## applying the framework
This framework assumes the existence of the below data-groups.  Here are sample set-up commands to get off the ground:
```
    tmsh create ltm data-group internal global_config { type string records add { hsl_enabled { data false } hsl_slow_response_threshold { data 1000 } hsl_errlog_404 { data false } } }

    tmsh create ltm data-group internal hsl_log_headers { type string records add { / } }

    tmsh create ltm data-group internal hsl_general_logger { type string records add { / } }

    tmsh create ltm data-group internal hsl_errlog { type string records add { / } }

    tmsh create ltm data-group internal preferred_client_ciphers { type string records add { AES128-GCM-SHA256 { } AES256-GCM-SHA384 { } AES256-SHA256 { } ECDHE-RSA-AES128-GCM-SHA256 { } ECDHE-RSA-AES256-GCM-SHA384 { } ECDHE-RSA-CHACHA20-POLY1305-SHA256 { } TLS13-AES128-GCM-SHA256 { } TLS13-AES256-GCM-SHA384 { } TLS13-CHACHA20-POLY1305-SHA256 { } } }

    tmsh create ltm data-group internal preferred_server_ciphers { records add { AES128-GCM-SHA256 { } AES128-SHA { } AES128-SHA256 { } AES256-GCM-SHA384 { } AES256-SHA { } AES256-SHA256 { } ECDHE-ECDSA-AES128-GCM-SHA256 { } ECDHE-ECDSA-AES128-SHA { } ECDHE-ECDSA-AES128-SHA256 { } ECDHE-ECDSA-AES256-GCM-SHA384 { } ECDHE-ECDSA-AES256-SHA { } ECDHE-ECDSA-AES256-SHA384 { } ECDHE-RSA-AES128-CBC-SHA { } ECDHE-RSA-AES128-GCM-SHA256 { } ECDHE-RSA-AES128-SHA256 { } ECDHE-RSA-AES256-CBC-SHA { } ECDHE-RSA-AES256-GCM-SHA384 { } ECDHE-RSA-AES256-SHA384 { } ECDHE-RSA-CHACHA20-POLY1305-SHA256 { } TLS13-AES128-GCM-SHA256 { } TLS13-AES256-GCM-SHA384 { } } type string }
```

A VIP should have either the `hsl_tcp` or `hsl_http` irule applied to log HSL events.  For HTTP vips, you can additionally add `http_ssl` if you have SSL profiles applied, or `hsl_cache` if you have a caching profile applied.  These will enrich the log events defined in `hsl_http`.

## event catalog
Here is a quick description of application logging events exposed out-of-the-box:

| Event Name    | slow_response               |
| :---          | :---              |
| valid stages  | HTTP_RESPONSE             |
| severity      | warning                  |
| description   | triggered by default when a request exceeds the response time threshold defined in the `global_config` datagroup                 |
| custom fields | n/a            |

| Event Name    | server_error             |
| :---          | :---              |
| stages        | HTTP_RESPONSE                  |
| severity      | error                  |
| description   | triggered for http status >= 400 if error logging is enabled in the datagroup `hsl_errlog`                  |
| custom fields | n/a                  |

| Event Name    | quick_log               |
| :---          | :---              |
| stages        | HTTP_RESPONSE, CLIENT_CLOSED, CACHE_RESPONSE |
| severity      | info                  |
| description   | can be triggered in an irule by setting `set hsl_quick_log 1` to add a log line to remote logs                  |
| custom fields | quick_log.data                  |

| Event Name    | general             |
| :---          | :---              |
| stages        | HTTP_RESPONSE, CLIENT_CLOSED, CACHE_RESPONSE                  |
| severity      | info                  |
| description   | logs traffic as defined in the datagroup `hsl_general_logger`                  |
| custom fields | n/a                  |

| Event Name    | lb_failed                |
| :---          | :---              |
| stages        | HTTP_RESPONSE, CLIENT_CLOSED                  |
| severity      | error                  |
| description   | triggered anytime the `LB_FAILED` event is triggered on a request; processes at the end of a successful retry OR client close      |
| custom fields | lb_failed.original_pool, lb_failed.failed_member, lb_failed.ltm_time_lbfail, lb_failed.original_pool_active_members      |

| Event Name    | weak_cipher             |
| :---          | :---              |
| stages        | HTTP_RESPONSE, CLIENT_CLOSED, CACHE_RESPONSE                  |
| severity      | error                  |
| description   | log triggered via irule `hsl_ssl` when client or cert handshake users ciphers not defined in the datagroups `preferred_client_ciphers` or `preferred_server_ciphers`  |
| custom fields |  weak_cipher.ssl_client_cipher, weak_cipher.ssl_client_version, weak_cipher.ssl_client_bits, weak_ciphers.ssl_server_cipher, weak_cipher.ssl_server_version, weak_cipher.ssl_server_bits  |

| Event Name    | no_response            |
| :---          | :---              |
| stages        | CLIENT_CLOSED                |
| severity      | error                  |
| description   | triggered when no HTTP_RESPONSE event is processed for a request                  |
| custom fields | n/a                  |

| Event Name    | custom_event    |
| :---          | :---              |
| stages        |                   |
| severity      |                   |
| description   | This is just a template for whatever custom events you would like to integrate with your own infrastructure!  Does not trigger as-is. |
| custom fields |                   |
