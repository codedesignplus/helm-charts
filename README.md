#ms-base-chart:
virtualService: 
  create: true
  hosts:
    - "services.codedesignplus.com"
  gateways:
    - services-gateway
  http:
  - route:
    - destination:
        host: hello-world
        port:
          number: 80

helm show values charts/.....tgz