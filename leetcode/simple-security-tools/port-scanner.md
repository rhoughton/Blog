# Port Scanner

A simple port scanner that takes user input for the host to be scanned and outputs the ports that are open

{% code title="port_scan.py" %}
```python
from socket import *

if __name__ == '__main__':
  target = input("Enter host to be scanned:")
  target_ip = gethostbyname(target)
  for i in range(65535):
    s = socket(AF_INET, SOCK_STREAM)
    con = s.connect_ex((target_ip, i))
    if (con == 0):
        print("Port {} is open".format(i))
    s.close()
```
{% endcode %}
