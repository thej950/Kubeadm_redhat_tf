This Terraform setup is structured to automate the deployment of EC2 instances configured with `kubeadm` on AWS, using modular and reusable components. Here's a breakdown of the provided directory and configuration:

---
```bash
vagrant@thej-machine:~$ tree Kubeadm_redhat_tf/
Kubeadm_redhat_tf/
├── kubeadm
│   └── main.tf
├── main.tf
├── provider.tf
├── readme.md
├── security-groups
│   └── main.tf
├── shell_scripts
│   ├── k8s.sh
│   └── kubeadm_script.sh
├── sshkeys
│   ├── aws-keys
│   └── aws-keys.pub
├── terraform.tfvars
└── variables.tf

4 directories, 11 files
vagrant@thej-machine:~$
```

### **Directory Structure**
1. **`kubeadm_redhat_tf/`**:
   - Main project directory containing all the Terraform files and related resources.

2. **Subdirectories**:
   - **`kubeadm/`**: Contains a `main.tf` file to define resources related to EC2 instances with `kubeadm`.
   - **`security-groups/`**: Defines a security group allowing all inbound and outbound traffic.
   - **`shell_scripts/`**: Contains shell scripts for Kubernetes (`k8s.sh`) and `kubeadm` setup (`kubeadm_script.sh`).
   - **`sshkeys/`**: Contains the private and public key files for SSH access (`aws-keys` and `aws-keys.pub`).

3. **Files**:
   - **`main.tf`**: Core Terraform configuration using modules to create resources like EC2 instances and security groups.
   - **`provider.tf`**: Specifies the AWS provider configuration.
   - **`variables.tf`**: Declares input variables for reusability.
   - **`terraform.tfvars`**: Provides values for declared variables.
   - **`readme.md`**: Likely contains documentation about the project.

---

### **Code Explanation**

#### **`provider.tf`**
- Defines the AWS provider:
  - **`region`**: `us-east-1` specifies the AWS region.
  - **`profile`**: Uses the AWS CLI profile `thej` for credentials.

#### **`main.tf`**
- **Modules**:
  - **`security_group`**: Creates a security group (`security-groups/main.tf`) to allow all traffic.
  - **`kubeadm`**: Launches 2 EC2 instances (`count = 2`) with:
    - AMI specified by `var.ec2_ami_id`.
    - Instance type `t2.medium`.
    - Public SSH key for access (`var.public_key`).
    - Security group from `security_group`.
    - A shell script (`kubeadm_script.sh`) passed as `user_data` to install `kubeadm`.

#### **`variables.tf`**
- **Variables**:
  - `ec2_ami_id`: Stores the AMI ID for EC2 instances.
  - `public_key`: Stores the SSH public key.

#### **`terraform.tfvars`**
- Specifies actual values for variables:
  - `public_key`: SSH public key for the EC2 instance.
  - `ec2_ami_id`: ID of the AMI used to launch instances.

#### **`kubeadm/main.tf`**
- Configures EC2 instances with:
  - AMI ID, instance type, key pair, and security group.
  - Tags and `user_data` for bootstrap scripts.
- **Outputs**:
  - `kubeadm_ec2_instance_ip`: Instance ID.
  - `dev_proj_1_ec2_instance_public_ip`: Public IP.

#### **`security-groups/main.tf`**
- **Security Group**:
  - Ingress: Allows all inbound traffic (`0.0.0.0/0`).
  - Egress: Allows all outbound traffic (`0.0.0.0/0`).
- **Output**:
  - Exposes the ID of the created security group for use by other modules.

#### **`shell_scripts/`**
- Contains scripts to:
  - Install Kubernetes components (`k8s.sh`).
  - Set up `kubeadm` on the launched EC2 instances (`kubeadm_script.sh`).

---

### **Working Flow**
1. **Initialize**:
   - Run `terraform init` to download necessary plugins and initialize the setup.

2. **Plan**:
   - Run `terraform plan` to preview the resources Terraform will create.

3. **Apply**:
   - Run `terraform apply` to launch:
     - 2 EC2 instances configured with `kubeadm`.
     - A security group allowing all traffic.

4. **Outputs**:
   - Public IPs and instance IDs will be displayed after successful execution.

---

### **Purpose**
- This setup automates Kubernetes cluster preparation using EC2 instances, `kubeadm`, and Terraform.
- The modular approach ensures reusability and scalability.
- The use of `user_data` and `shell_scripts` simplifies provisioning and configuration.

===
The `variable` declarations like `variable "ami_id" {}` and others are used to define **input variables** in Terraform. These variables make the configuration more **dynamic**, **reusable**, and **easy to manage**. Here's why each is used in this context:

---

### **Purpose of Each Variable**

1. **`variable "ami_id" {}`**:
   - Specifies the Amazon Machine Image (AMI) ID to use for the EC2 instances.
   - Instead of hardcoding the AMI ID, this variable allows flexibility to use different AMIs by just passing a value in `terraform.tfvars`.

   **Example**:
   ```hcl
   ami_id = "ami-0fe630eb857a6ec83"
   ```

2. **`variable "instance_type" {}`**:
   - Specifies the type of EC2 instance to launch (e.g., `t2.medium`, `t3.large`).
   - Using a variable allows you to easily switch between instance types without modifying the resource definition.

   **Example**:
   ```hcl
   instance_type = "t2.medium"
   ```

3. **`variable "tag_name" {}`**:
   - Used to define a custom tag name for the EC2 instance, such as `"kubeadm:Redhat Linux EC2"`.
   - Tags are crucial for identifying and managing resources within AWS.

   **Example**:
   ```hcl
   tag_name = "kubeadm:Redhat Linux EC2"
   ```

4. **`variable "public_key" {}`**:
   - Stores the public SSH key used to enable secure login to the EC2 instance.
   - The `aws_key_pair` resource references this variable to create an AWS Key Pair.

   **Example**:
   ```hcl
   public_key = "ssh-rsa AAAAB3Nza..."
   ```

5. **`variable "sg_for_kubeadm" {}`**:
   - Holds the IDs of security groups that will be attached to the EC2 instance.
   - This allows the instance to have specific network permissions, such as inbound and outbound traffic rules.

   **Example**:
   ```hcl
   sg_for_kubeadm = [module.security_group.sg_ec2_sg_allow_all_id]
   ```

6. **`variable "enable_public_ip_address" {}`**:
   - Determines whether the EC2 instance should have a public IP address.
   - Setting this as a variable allows you to toggle public IP allocation easily.

   **Example**:
   ```hcl
   enable_public_ip_address = true
   ```

7. **`variable "user_data_install_kubeadm" {}`**:
   - Contains the user data script to bootstrap the instance with specific software (`kubeadm` in this case).
   - By making it a variable, you can change the bootstrap script dynamically without altering the resource.

   **Example**:
   ```hcl
   user_data_install_kubeadm = templatefile("./shell_scripts/kubeadm_script.sh", {})
   ```

---

### **Why Use Variables?**

1. **Flexibility**:
   - The configuration can be reused across different environments (e.g., dev, test, prod) by simply providing different variable values.

2. **Maintainability**:
   - Instead of modifying multiple places in your code, you just update the `terraform.tfvars` or pass variables directly in the command.

3. **Readability**:
   - It makes the Terraform code cleaner and easier to understand by separating the logic from the data.

4. **Reusability**:
   - The same Terraform modules can be used for different projects or configurations by passing different values to the variables.

---

### **How They Work Together**

- These variables are referenced in the `aws_instance` resource to dynamically assign values for instance attributes such as AMI ID, instance type, security group, and public IP.
- Values for these variables are provided in `terraform.tfvars` or during runtime using the `-var` flag.
