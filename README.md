# nginx-unit

Конфигурация Nginx-Unit для Битрикс

bitrix.json
```
{
    "listeners": {
        "127.0.0.1:8090": {
            "application": "bitrix_urlrewrite"
        },
	"127.0.0.1:8091": {
            "application": "bitrix_direct"
        }
    },
    "applications": {
        "bitrix_urlrewrite": {
            "type": "php",
              "processes": {
               "max": 20,
               "spare": 5
              },
            "user": "bitrix",
            "group": "bitrix",
            "root": "/home/bitrix/www/",
            "script": "bitrix/urlrewrite.php",
            "options": {
                "file": "/etc/php.ini",
                "admin": {
                    "memory_limit": "1024M",
                    "variables_order": "EGPCS",
                    "expose_php": "0"
                },
                "user": {
                    "display_errors": "1"
                }
            }
	},
	"bitrix_direct": {
            "type": "php",
            "processes": 20,
            "user": "bitrix",
            "group": "bitrix",
            "root": "/home/bitrix/www/",
            "index": "index.php",
            "options": {
                "file": "/etc/php.ini",
                "admin": {
                    "memory_limit": "1024M",
                    "variables_order": "EGPCS",
                    "expose_php": "0"
                },
                "user": {
                    "display_errors": "1"
                }
            }
	}
    },
    "access_log": "/var/log/unit/access.log"
}
```

`curl -X PUT -d @bitrix.json --unix-socket /run/unit/control.sock http://localhost/config`

Конфигурация для Nginx:

```
upstream unit_bitrix_urlrewrite {
    server 127.0.0.1:8090;
}
upstream unit_bitrix_direct {
    server 127.0.0.1:8091;
}
```

```
    location = / {
        proxy_pass       http://unit_bitrix_direct;
        proxy_redirect   http://unit_bitrix_direct /;
        proxy_read_timeout 60s;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location / {
        try_files $uri $uri/ @urlrewrite_php;
    }
    location @urlrewrite_php {
        proxy_pass       http://unit_bitrix_urlrewrite;
        proxy_redirect   http://unit_bitrix_urlrewrite /;
        proxy_read_timeout 60s;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location ~* \.php$ {
        try_files        $uri =404;
        proxy_pass	 http://unit_bitrix_direct;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
```
