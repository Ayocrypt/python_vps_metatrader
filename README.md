
# MT5 Multi-Account Trading Server Setup

## VPS Requirements
- Provider: Vultr/ForexVPS.net
- OS: Windows Server 2019/2022
- Minimum Specs:
  - CPU: 2 cores
  - RAM: 4GB
  - Storage: 60GB
  - Network: 1Gbps

## Installation Steps

### 1. Access VPS
```powershell
# Connect via web console or RDP
# Ensure PowerShell is running as Administrator
```

### 2. Install Python 3.9
```powershell
# Create downloads directory
mkdir C:\Downloads
cd C:\Downloads

# Download Python 3.9
Invoke-WebRequest -Uri "https://www.python.org/ftp/python/3.9.13/python-3.9.13-amd64.exe" -OutFile "python39.exe"

# Install Python (with PATH option)
.\python39.exe /quiet InstallAllUsers=1 PrependPath=1

# Verify installation
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
python --version
pip --version
```

### 3. Install MT5
```powershell
# Download MT5
Invoke-WebRequest -Uri "https://download.mql5.com/cdn/web/metaquotes.software.corp/mt5/mt5setup.exe" -OutFile "mt5setup.exe"

# Install MT5
.\mt5setup.exe /auto

# Wait for installation to complete
Start-Sleep -Seconds 30
```

### 4. Install Required Python Packages
```powershell
# Install packages
pip install MetaTrader5==5.0.37 flask==2.0.1 numpy==1.24.3

# Verify installations
pip list | findstr "MetaTrader5 flask numpy"
```

## Testing Setup

### 1. Basic MT5 Connection Test
```powershell
# Create test script
@"
import MetaTrader5 as mt5
print('MT5 Package Version:', mt5.__version__)

if mt5.initialize():
    print('MT5 Terminal Version:', mt5.version())
    mt5.shutdown()
else:
    print('MT5 Initialize failed')
"@ | Out-File -FilePath "test_mt5.py" -Encoding utf8

# Run test
python test_mt5.py
```

### 2. Account Connection Test
```powershell
# Create account test script
@"
import MetaTrader5 as mt5

# Demo account details
DEMO_LOGIN = YOUR_LOGIN
DEMO_PASSWORD = "YOUR_PASSWORD"
DEMO_SERVER = "MetaQuotes-Demo"

print('Testing MT5 Account Connection...')
if mt5.initialize(
    login=DEMO_LOGIN,
    password=DEMO_PASSWORD,
    server=DEMO_SERVER
):
    account_info = mt5.account_info()
    if account_info is not None:
        print(f'Login: {account_info.login}')
        print(f'Balance: ${account_info.balance}')
        print(f'Equity: ${account_info.equity}')
    mt5.shutdown()
"@ | Out-File -FilePath "test_account.py" -Encoding utf8

# Run account test
python test_account.py
```

### 3. Multi-Account Test
```powershell
# Create multi-account test script
@"
import MetaTrader5 as mt5
import time
from datetime import datetime

ACCOUNTS = [
    {
        "login": ACCOUNT1_LOGIN,
        "password": "ACCOUNT1_PASSWORD",
        "server": "MetaQuotes-Demo",
        "name": "Account1"
    },
    # Add more accounts as needed
]

def check_account(account):
    print(f"\nChecking {account['name']}...")
    if mt5.initialize(
        login=account['login'],
        password=account['password'],
        server=account['server']
    ):
        info = mt5.account_info()
        print(f"Balance: ${info.balance}")
        mt5.shutdown()
        return True
    return False

for account in ACCOUNTS:
    check_account(account)
    time.sleep(1)
"@ | Out-File -FilePath "test_multi_account.py" -Encoding utf8

# Run multi-account test
python test_multi_account.py
```

## Troubleshooting

### Common Issues
1. MT5 initialization fails:
   - Ensure MT5 terminal is installed properly
   - Check account credentials
   - Verify server name

2. Python package installation fails:
   - Update pip: `python -m pip install --upgrade pip`
   - Install Visual C++ Build Tools if required

3. Connection issues:
   - Check internet connection
   - Verify firewall settings
   - Ensure MT5 is not blocked

### Useful Commands
```powershell
# Check MT5 process
Get-Process | Where-Object {$_.Name -like "*terminal64*"}

# Check Python installation
where python

# Check installed packages
pip list

# Check system encoding
[System.Text.Encoding]::Default
```

## Next Steps
1. Set up automated trading
2. Configure multiple accounts
3. Implement monitoring system
4. Set up error handling
5. Create backup procedures

## Security Notes
- Keep credentials secure
- Use strong passwords
- Regularly update systems
- Monitor access logs
- Implement proper authentication

