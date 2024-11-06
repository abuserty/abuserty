# CLOUD WATCH AGENT SETUP

## Cloud Watch Package installation in Ubuntu Instance with dpkg

```
#wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/arm64/latest/amazon-cloudwatch-agent.deb
#sudo dpkg -i ./amazon-cloudwatch-agent.deb
#sudo apt-get install -f
#sudo dpkg --configure -a
#sudo apt-get install -f
------------------------------------------------------------------------------------------------------------
#!/bin/bash

# Define the URL for the Amazon CloudWatch Agent package
PACKAGE_URL="https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/arm64/latest/amazon-cloudwatch-agent.deb"

# Define the package name
PACKAGE_NAME="amazon-cloudwatch-agent.deb"

# Download the package
echo "Downloading Amazon CloudWatch Agent..."
wget $PACKAGE_URL -O $PACKAGE_NAME

# Install the package
echo "Installing Amazon CloudWatch Agent..."
sudo dpkg -i ./$PACKAGE_NAME

# Fix any missing dependencies
echo "Fixing dependencies..."
sudo apt-get install -f -y

# Configure any unconfigured packages
echo "Configuring packages..."
sudo dpkg --configure -a

# Run apt-get install -f again to ensure all dependencies are resolved
echo "Running apt-get install -f to ensure all dependencies are resolved..."
sudo apt-get install -f -y

# Clean up the downloaded package file
echo "Cleaning up..."
rm -f ./$PACKAGE_NAME

echo "Installation completed."
 #Service Status check
 systemctl status amazon-cloudwatch-agent
```

### This is the Command to setup Agents

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

Manually Config for Disk & Memory utilization

```
#!/bin/bash

# Change to the directory where the CloudWatch Agent configuration file is located
cd /opt/aws/amazon-cloudwatch-agent/bin

# Backup the existing configuration file
cp config.json config.json.bak

# Overwrite the existing configuration file with new settings
cat <<EOL > config.json
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "root"
  },
  "metrics": {
    "namespace": "CWAgent",
    "append_dimensions": {
      "InstanceId": "\${aws:InstanceId}"
    },
    "metrics_collected": {
      "mem": {
        "measurement": [
          "mem_used_percent"
        ],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": [
          "disk_used_percent"
        ],
        "metrics_collection_interval": 70,
        "resources": [
          "/",
          "/data",
          "/log"
        ]
      }
    }
  }
}
EOL

# Fetch the new configuration and start the CloudWatch Agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s

# Enable the CloudWatch Agent service to start on boot
sudo systemctl enable amazon-cloudwatch-agent

# Check the status of the CloudWatch Agent service
sudo systemctl status amazon-cloudwatch-agent
```
