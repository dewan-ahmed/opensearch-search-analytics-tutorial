## Generate mock data

The following Python code generates 1000 mock documents using Python faker libary and adds the data to a JSON file. A sample document looks like this:

```json
{
        "server_id": "8297def4-7fca-41f4-9368-e48c3c6b24c3",
        "server_ip": "66.125.35.51",
        "server_mac": "c9:87:2a:b0:ab:f3",
        "server_location": "Toddburgh",
        "server_memory_used": 78,
        "server_temperature": 63,
        "server_os": "Fedora",
        "server_start_time": "03/18/2023 08:58:12",
        "hardware": {
            "type": "Memory",
            "capacity": 1,
            "vendor": "Nelson-Morton"
        },
        "security": {
            "firewall_rules": [
                {
                    "port": 5802,
                    "protocol": "UDP",
                    "access": "DENY"
                },
                {
                    "port": 53825,
                    "protocol": "TCP",
                    "access": "ALLOW"
                },
                {
                    "port": 2810,
                    "protocol": "TCP",
                    "access": "DENY"
                },
                {
                    "port": 9045,
                    "protocol": "UDP",
                    "access": "DENY"
                },
                {
                    "port": 62976,
                    "protocol": "UDP",
                    "access": "ALLOW"
                }
            ],
            "security_events": [
                {
                    "type": "Login Attempt",
                    "date": "06/05/2023 12:41:32"
                },
                {
                    "type": "Intrusion Detection",
                    "date": "06/10/2023 21:27:32"
                },
                {
                    "type": "Authentication Failure",
                    "date": "06/18/2023 03:41:03"
                },
                {
                    "type": "Login Attempt",
                    "date": "06/11/2023 02:52:30"
                },
                {
                    "type": "Authentication Failure",
                    "date": "06/05/2023 10:37:16"
                },
                {
                    "type": "Intrusion Detection",
                    "date": "06/10/2023 18:21:06"
                }
            ]
        }
    }
```

Let's generate the sample data:

```python
import json
import random
from faker import Faker

fake = Faker()

# Generate nested hardware data
def generate_hardware_data():
    hardware_type = random.choice(['CPU', 'Memory', 'Storage'])
    hardware_capacity = random.randint(1, 10)
    hardware_vendor = fake.company()
    hardware_data = {
        "type": hardware_type,
        "capacity": hardware_capacity,
        "vendor": hardware_vendor
    }
    return hardware_data

# Generate nested security data
def generate_security_data():
    firewall_rules = []
    for _ in range(random.randint(1, 5)):
        port = random.randint(1, 65535)
        protocol = random.choice(['TCP', 'UDP'])
        access = random.choice(['ALLOW', 'DENY'])
        firewall_rule = {
            "port": port,
            "protocol": protocol,
            "access": access
        }
        firewall_rules.append(firewall_rule)

    security_events = []
    for _ in range(random.randint(1, 10)):
        event_type = random.choice(['Login Attempt', 'Authentication Failure', 'Intrusion Detection'])
        event_date = fake.date_time_this_month().strftime('%m/%d/%Y %H:%M:%S')
        security_event = {
            "type": event_type,
            "date": event_date
        }
        security_events.append(security_event)

    security_data = {
        "firewall_rules": firewall_rules,
        "security_events": security_events
    }
    return security_data

# Generate 1000 server log entries
server_logs = []
for _ in range(1000):
    server_id = fake.uuid4()
    server_ip = fake.ipv4()
    server_mac = fake.mac_address()
    server_location = fake.city()
    server_memory_used = random.randint(0, 100)
    server_temperature = random.randint(20, 85)
    server_os = random.choice(['RHEL', 'Fedora', 'FreeBSD', 'Windows Server 2003', 'Windows Server 2008', 'Windows Server 2012', 'Windows Server 2016', 'Windows Server 2019', 'Ubuntu', 'Debian', 'CentOS', 'SUSE', 'openSUSE', 'Arch Linux', 'Gentoo', 'Slackware', 'Linux Mint', 'Elementary OS', 'Kali Linux'])
    server_start_time = fake.date_time_this_decade().strftime('%m/%d/%Y %H:%M:%S')
    hardware_data = generate_hardware_data()
    security_data = generate_security_data()

    server_log = {
        "server_id": server_id,
        "server_ip": server_ip,
        "server_mac": server_mac,
        "server_location": server_location,
        "server_memory_used": server_memory_used,
        "server_temperature": server_temperature,
        "server_os": server_os,
        "server_start_time": server_start_time,
        "hardware": hardware_data,
        "security": security_data
    }
    server_logs.append(server_log)

# Write server log entries to a JSON file
with open('server-logs.json', 'w') as file:
    json.dump(server_logs, file, indent=4)
```
