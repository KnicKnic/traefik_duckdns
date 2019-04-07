# traefik_duckdns
Short example of setting up TLS end to end using traefik & duckdns for free wildcard subdomain & dynamic ip support for a website. 

Websites `https://*.example.duckdns.org` and `https://example.duckdns.org` 

Useful for
1. You want simple certs for an API or website
1. Server on a home internet with an ISP that changes your IP.
1. Don't want to or can't expose port 80 or even 443.
    1. This can be useful for a computer on a private network that has internet through NAT or proxy.
        1. I use it to secure my private internal docker registry

## Steps
1. copy this repository into the current directory
    1. Linux - `docker run --rm -v $(pwd):/data -u "${UID}" busybox sh -c "wget -nc -O - https://github.com/KnicKnic/traefik_duckdns/archive/master.zip | unzip - -d /tmp && cp -r /tmp/traefik_duckdns-master/* /data/ && cp -r /tmp/traefik_duckdns-master/.??* /data/"` 
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

## Additional steps (may be appropriate)
1. ensure web traffic (https port 443, optionally http port 80) can get to your server
    1. http://portforward.com
1. add / modify forwarding rules
    1. if you just want to change `https://subdomain.duckdns.org` to point to your backend, modify rules/nginx.toml
        1. replace `url = "http://nginx:80"` with your backend url
    1. copy and modify sample rules/nginx.toml into rules/copy.toml
        1. replace every instance of `nginx` with `copy` or some other word
        1. replace `url = "http://nginx:80"` with your backend url
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
1. Allow traefik to work in proxied / firewall network
    1. You will probably want to do 2 things
        1. Complete cert challenges by purely delaying dns checks.
            1. in config/traefik.toml uncomment delayBeforeCheck
            1. Do this if DNS queries are blocked
        1. Not auto update the IP
            1. in docker-compose.yml comment out the whole updateDuckDNSIP section
            1. If you are exposing a private computer that is not routable we do not want to override it with our pubic NAT address
                1. Useful in scenarios where you only want to access the website inside a corprate network (useful for TLS for private docker registries)
                1. You must manually set the ip for the domain name in duckdns.org
1. Write a project that extends this one.
    1. example: https://github.com/KnicKnic/myq-garage-server
1. Learn how to have traefik watch new containers so you do not need to update rules
    1. I only wrote about file based rules, I did this for a few reasons
        1. It works in non-container scenarios you can reference any ip, or hostname for the backend
        1. Security / Multiplatform, you do not have to worry about security getting the docker socket, or how to read the docker socket in windows
    1. read more about adding labels and auto configuring traefik here
        1. https://medium.com/@joshuaavalon/setup-traefik-step-by-step-406792afe9b2
    
