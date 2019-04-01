# traefik_duckdns
Short example of setting up TLS end to end using traefik & duckdns for free wildcard subdomain & dynamic ip support for a website.

## Steps
1. copy this repository into the current directory
    1. Linux - `wget -nc -O traefik_duckdns-github.zip https://github.com/KnicKnic/traefik_duckdns/archive/master.zip && unzip -n traefik_duckdns-github.zip -d . && cp -rn traefik_duckdns-master/* . && cp -rn traefik_duckdns-master/.??* .` 
    1. Windows - `curl.exe -L -o traefik_duckdns-github.zip https://github.com/KnicKnic/traefik_duckdns/archive/master.zip ;
 Expand-Archive -DestinationPath . traefik_duckdns-github.zip ;  cp -r .\traefik_duckdns-master\* .`
1. ensure you have docker & docker-compose installed
    1. https://docs.docker.com/compose/install/
1. get a duckdns subdomain
    1. go to http://duckdns.org & click one of the sign in options across the top
    1. sign up for a subdomain
1. update .env file with subdomain and token from previous step
    1. Token is a field you can see when signed in seeing the subdomains
1. start everything by running `docker-compose up -d`
    1. be patient it can take up to 5 minutes for certs to get populated (until then you will get untrusted cert)
    1. test by going to `https://subdomain.duckdns.org` you should see the nginx banner

## Additional steps
1. ensure webtraffic (https port 443, optionally http port 80) can get to your server
    1. http://portforward.com
1. add more forwarding rules
    1. copy and modify sample rules/nginx.toml into rules/copy.toml
        1. replace every instance of `nginx` with `copy` or some other word
        1. repace `url = "http://nginx:80"` with your backend url
        1. replace `rule = "HostRegexp:www.{subdomain:[^.]+}.duckdns.org,{subdomain:[^.]+}.duckdns.org"` with your new name like `rule = "HostRegexp:copy.{subdomain:[^.]+}.duckdns.org"`
        1. read more info from traefik 
            1. https://docs.traefik.io/basics/#frontends
            1. https://docs.traefik.io/basics/#backends
            1. https://docs.traefik.io/configuration/backends/file/
1. enabled traefik dashboard
    1. edit rules/traefik.toml
        1. uncomment everything (only remove 1 # per line)
        1. update password as explained in the file
    1. go to `https://traefik.subdomain.duckdns.org` and enter username & password
    
