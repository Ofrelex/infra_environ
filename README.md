# Understanding Environment Variables &amp; Infrastructure Environments: Key Differences

In this mini project, I gained a deeper understanding of how infrastructure environments and environment variables work together to make software development, testing, and deployment more structured, efficient, and secure. Infrastructure environments refer to the different stages of the software lifecycle—local (development), testing, and production—each providing a controlled setup for building, experimenting, validating, and finally delivering applications to end users. For example, local development might happen on a personal machine using VirtualBox and Ubuntu, while testing and production environments may be hosted on separate AWS accounts with EC2 instances. These distinct environments ensure that code is developed safely, tested under realistic conditions, and deployed securely for customer use.

On the other hand, environment variables are powerful tools that allow scripts and applications to behave dynamically depending on the environment in which they are running. Instead of hardcoding values like database URLs, usernames, or passwords, which would limit flexibility and pose security risks, environment variables provide a way to switch configurations seamlessly. For instance, the same script can connect to a local database in development, a test database in the testing phase, and a live production database in the production phase—simply by reading the correct variables set for that environment.

By developing the aws_cloud_manager.sh script, I learned how to implement these concepts practically. The script demonstrates how to check environment variables, handle missing or invalid inputs, and use positional parameters to pass arguments dynamically at runtime. This means instead of editing the script manually each time, I can simply run commands like ./aws_cloud_manager.sh testing or ./aws_cloud_manager.sh production 5, and the script automatically adjusts to the correct environment and even scales based on arguments like the number of instances. I also learned the importance of input validation and error handling, such as ensuring the correct number of arguments are passed before execution, which helps prevent bugs and provides clear instructions to users.

Overall, this project emphasized the importance of writing clean, reusable, and flexible scripts that adapt to multiple environments without repetitive changes. It taught me why hardcoding values is poor practice, how environment variables safeguard sensitive information, and how positional parameters give scripts the flexibility to scale. These lessons are not only applicable in shell scripting but are also foundational skills for cloud infrastructure management, particularly in environments like AWS where managing multiple environments and configurations is essential for smooth and secure operations.

Here’s a complete version of the aws_cloud_manager.sh script that:

Accepts positional arguments (environment + number of instances).

Uses the $ENVIRONMENT variable dynamically.

Integrates with the AWS CLI (for EC2 provisioning, listing instances, etc.).

Includes argument validation and error handling.

```
#!/bin/bash
# aws_cloud_manager.sh
# Script to manage AWS EC2 instances in different environments (local, testing, production)

# --- Step 1: Validate number of arguments ---
if [ "$#" -lt 2 ]; then
    echo "Usage: $0 <environment: local|testing|production> <number_of_instances>"
    exit 1
fi

# --- Step 2: Assign positional parameters ---
ENVIRONMENT=$1
NUMBER_OF_INSTANCES=$2

# --- Step 3: Print environment info ---
echo "Selected Environment: $ENVIRONMENT"
echo "Number of instances to provision: $NUMBER_OF_INSTANCES"

# --- Step 4: Configure environment variables ---
if [ "$ENVIRONMENT" == "local" ]; then
    DB_URL="localhost"
    DB_USER="test_user"
    DB_PASS="test_pass"
    AWS_REGION="us-east-1"
    AMI_ID="ami-1234567890abcdef0" # replace with a dummy/local image
    INSTANCE_TYPE="t2.micro"

elif [ "$ENVIRONMENT" == "testing" ]; then
    DB_URL="testing-db.example.com"
    DB_USER="testing_user"
    DB_PASS="testing_pass"
    AWS_REGION="us-east-1"
    AMI_ID="ami-0abcdef1234567890" # replace with a valid test AMI
    INSTANCE_TYPE="t2.micro"

elif [ "$ENVIRONMENT" == "production" ]; then
    DB_URL="production-db.example.com"
    DB_USER="prod_user"
    DB_PASS="prod_pass"
    AWS_REGION="us-east-1"
    AMI_ID="ami-0abcdef1234567890" # replace with a valid production AMI
    INSTANCE_TYPE="t2.small"

else
    echo "Invalid environment specified. Please use local, testing, or production."
    exit 2
fi

# --- Step 5: Provision EC2 Instances ---
echo "Provisioning $NUMBER_OF_INSTANCES EC2 instance(s) in $ENVIRONMENT environment..."

aws ec2 run-instances \
    --image-id $AMI_ID \
    --count $NUMBER_OF_INSTANCES \
    --instance-type $INSTANCE_TYPE \
    --region $AWS_REGION \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Environment,Value=$ENVIRONMENT}]"

if [ $? -eq 0 ]; then
    echo "Successfully started $NUMBER_OF_INSTANCES EC2 instance(s) in $ENVIRONMENT."
else
    echo "Error: Failed to provision EC2 instance(s)."
    exit 3
fi

# --- Step 6: List running instances in the environment ---
echo "Listing running EC2 instances for $ENVIRONMENT..."
aws ec2 describe-instances \
    --filters "Name=tag:Environment,Values=$ENVIRONMENT" \
    --query "Reservations[*].Instances[*].{ID:InstanceId,State:State.Name}" \
    --region $AWS_REGION \
    --output table
```
