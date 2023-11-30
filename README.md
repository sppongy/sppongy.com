<a style="color:white" href="https://sppongy.com"><h1 align="center"><sup>S</sup>ppongy<sub>.com</sub></h1></a>
<p align="center"> I'll put everything in a wiki eventually</p>

### Temp - Manga
Auto downloading manga is a pain so i manually fill my backlog and use RSS for new chapters  
qBittorrent has support for RSS feeds and auto download rules so that paired with nyaa.si's rss feeds it should work, (untested for now but looks like it'll work)  
So honestly just check nyaa and find the format new chapters are usually posted in and set the rss rules to filter that :thumb:  

### Containers (out of date)
- <a style="color:white" href="https://github.com/NginxProxyManager/nginx-proxy-manager">Nginx Proxy Manager</a>
- <a style="color:white" href="https://github.com/linuxserver/docker-wireguard">Wireguard</a>
- <a style="color:white" href="https://github.com/dani-garcia/vaultwarden">Vaultwarden</a>
- <a style="color:white" href="https://github.com/jellyfin/jellyfin">Jellyfin</a>
- <a style="color:white" href="https://github.com/Fallenbagel/jellyseerr">Jellyseerr</a>
- <a style="color:white" href="https://github.com/Kareadita/Kavita">Kavita</a>
- <a style="color:white" href="https://github.com/Sonarr/Sonarr">Sonarr</a>
- <a style="color:white" href="https://github.com/Radarr/Radarr/">Radarr</a>
- <a style="color:white" href="https://github.com/Prowlarr/Prowlarr">Prowlarr</a>
- <a style="color:white" href="https://github.com/morpheus65535/bazarr">Bazarr</a>
- <a style="color:white" href="https://github.com/kiranshila/Doplarr">Doplarr</a>
- <a style="color:white" href="https://github.com/qbittorrent/qBittorrent">qBittorrent</a>
- <a style="color:white" href="https://github.com/sabnzbd/sabnzbd">SABnzbd</a>
- <a style="color:white" href="https://github.com/go-gitea/gitea">Gitea</a>
- <a style="color:white" href="https://github.com/benphelps/homepage">Homepage</a>

## Nginx Proxy Manager Authelia config
location /authelia {
    internal;
    set $upstream_authelia http://authelia:9091/api/verify;
    proxy_pass_request_body off;
    proxy_pass $upstream_authelia;    
    proxy_set_header Content-Length "";
 
    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
    client_body_buffer_size 128k;
    proxy_set_header Host $host;
    proxy_set_header X-Original-URL $scheme://$http_host$request_uri;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr; 
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $http_host;
    proxy_set_header X-Forwarded-Uri $request_uri;
    proxy_set_header X-Forwarded-Ssl on;
    proxy_redirect  http://  $scheme://;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_cache_bypass $cookie_session;
    proxy_no_cache $cookie_session;
    proxy_buffers 4 32k;
 
    send_timeout 5m;
    proxy_read_timeout 240;
    proxy_send_timeout 240;
    proxy_connect_timeout 240;
}
 
    location / {
	## CHANGE NAME AND PORT TO SERVICE + UPSTREAM
        set $upstream_jellyfin http://jellyfin:8096;  
        proxy_pass $upstream_jellyfin;
 
		auth_request /authelia;
		auth_request_set $target_url $scheme://$http_host$request_uri;
		auth_request_set $user $upstream_http_remote_user;
		auth_request_set $groups $upstream_http_remote_groups;
		proxy_set_header Remote-User $user;
		proxy_set_header Remote-Groups $groups;
		error_page 401 =302 https://auth.sppongy.com/?rd=$target_url; 
 
		client_body_buffer_size 128k;
 
		proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
 
		send_timeout 5m;
		proxy_read_timeout 360;
		proxy_send_timeout 360;
		proxy_connect_timeout 360;
 
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_set_header X-Forwarded-Host $http_host;
		proxy_set_header X-Forwarded-Uri $request_uri;
		proxy_set_header X-Forwarded-Ssl on;
		proxy_redirect  http://  $scheme://;
		proxy_http_version 1.1;
		proxy_set_header Connection "";
		proxy_cache_bypass $cookie_session;
		proxy_no_cache $cookie_session;
		proxy_buffers 64 256k;
 
		set_real_ip_from 192.168.1.0/16;
		real_ip_header X-Forwarded-For;
		real_ip_recursive on;
 
    }
