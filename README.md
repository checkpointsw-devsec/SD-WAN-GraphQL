# SD-WAN-GraphQL
make sure you have requirements installed on your host
```BASH
pip install -r requirements.txt
```

usage:
```BASH
join-sdwan-profile.py --gateway <gateway name> --profile "<profile name>"
```

example: 
```BASH
join-sdwan-profile.py --gateway hq-exl-1 --profile "SD-WAN Gateways"
```

options:
```BASH
join-sdwan-profile.py --gateway hq-exl --profile "SD-WAN Gateways" [--debug <reuest | response | both> [--env /path/to/.env]
```
