### prereq is to have ExternalDNS, cert-util & cert-manager running on OCP
## then apply everything
oc apply -f getwellsoon/
## then copy the img and the html
oc cp index.html webserver-57bcfddc45-4w5nz:/usr/share/nginx/html
oc cp IMG_0978.JPG webserver-57bcfddc45-4w5nz:/usr/share/nginx/html/IMG_0978.JPG