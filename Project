#!/bin/bash

# AWS Configuration
AWS_REGION="us-east-1"    # Replace with your desired region
INSTANCE_TYPE="t2.micro"  # Replace with your desired instance type
KEY_PAIR_NAME="your-key-pair-name"   # Replace with your existing key pair name in AWS
SECURITY_GROUP_NAME="ros-security-group"   # Replace with a unique name for the security group

# Create a new security group with SSH and ROS ports open
aws ec2 create-security-group --group-name $SECURITY_GROUP_NAME \
    --description "Security group for ROS instance"

# Allow SSH access from your IP (replace YOUR_IP_ADDRESS with your actual IP)
aws ec2 authorize-security-group-ingress --group-name $SECURITY_GROUP_NAME \
    --protocol tcp --port 22 --cidr YOUR_IP_ADDRESS/32

# Allow ROS communication ports (replace YOUR_IP_ADDRESS with your actual IP)
aws ec2 authorize-security-group-ingress --group-name $SECURITY_GROUP_NAME \
    --protocol tcp --port 11311 --cidr YOUR_IP_ADDRESS/32

# Launch an EC2 instance with Ubuntu AMI
INSTANCE_ID=$(aws ec2 run-instances --image-id ami-0c55b159cbfafe1f0 \
                --count 1 --instance-type $INSTANCE_TYPE \
                --key-name $KEY_PAIR_NAME --security-groups $SECURITY_GROUP_NAME \
                --region $AWS_REGION --query 'Instances[0].InstanceId' --output text)

echo "Waiting for the instance to start..."
aws ec2 wait instance-running --instance-ids $INSTANCE_ID

# Get the public IP address of the instance
PUBLIC_IP=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID \
                --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)





# SSH into the instance using the key pair (replace YOUR_KEY_PAIR.pem with your actual key pair)
ssh -i "YOUR_KEY_PAIR.pem" ubuntu@$PUBLIC_IP << EOF
    # Update the package list and install dependencies
    sudo apt-get update
    sudo apt-get install -y curl gnupg2 lsb-release

    # Set up sources and install ROS 2 Foxy
    curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
    sudo sh -c 'echo "deb [arch=amd64,arm64] http://packages.ros.org/ros2/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros2-latest.list'
    sudo apt-get update
    sudo apt-get install -y ros-foxy-desktop

    # Source ROS in .bashrc
    echo "source /opt/ros/foxy/setup.bash" >> ~/.bashrc

    # Install additional ROS tools (e.g., colcon, rosdep)
    sudo apt-get install -y python3-colcon-common-extensions python3-rosdep

    # Initialize rosdep and update
    sudo rosdep init
    rosdep update

    # Source ROS for the current session
    source /opt/ros/foxy/setup.bash

    # Install any additional ROS packages or simulation tools you need (e.g., Gazebo)

EOF

echo "ROS installed successfully on the instance with ID: $INSTANCE_ID"

# Install X11 and desktop environment on the cloud instance
ssh -i "YOUR_KEY_PAIR.pem" ubuntu@$PUBLIC_IP << EOF
    sudo apt-get install -y xserver-xorg xfce4

    # Allow X11 forwarding in SSH configuration
    echo "X11Forwarding yes" | sudo tee -a /etc/ssh/sshd_config
    echo "X11UseLocalhost no" | sudo tee -a /etc/ssh/sshd_config

    # Restart SSH service to apply the changes
    sudo service ssh restart

EOF

ssh -i "YOUR_KEY_PAIR.pem" ubuntu@$PUBLIC_IP


echo "Public IP Address: $PUBLIC_IP"
