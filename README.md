# Jenkins Setup and Troubleshooting on Ubuntu EC2

## Project Overview
This project involves setting up Jenkins on an Ubuntu EC2 instance. The process includes installing Jenkins, resolving issues during installation, and troubleshooting service startup errors. This README documents the steps, commands, and troubleshooting techniques used.

---

## Prerequisites
- **Ubuntu EC2 Instance**
- **Java (OpenJDK 11 or higher)**
- **Administrator privileges**

---

## Steps to Install and Configure Jenkins

### 1. Update System Packages
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Java
Jenkins requires Java to run. Install OpenJDK 11:
```bash
sudo apt install openjdk-11-jdk -y
```
Verify Java installation:
```bash
java -version
```

### 3. Add Jenkins Repository and Key
```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### 4. Install Jenkins
Update the package index and install Jenkins:
```bash
sudo apt update
sudo apt install jenkins -y
```

### 5. Start and Enable Jenkins Service
Start the Jenkins service:
```bash
sudo systemctl start jenkins
```
Enable Jenkins to start on boot:
```bash
sudo systemctl enable jenkins
```

### 6. Troubleshooting Service Failures
If Jenkins fails to start, check its status:
```bash
sudo systemctl status jenkins
```
Examine detailed logs:
```bash
sudo journalctl -xeu jenkins.service
```

### 7. Verify Installation
Access Jenkins through a web browser:
```
http://<your-ec2-public-ip>:8080
```
Retrieve the initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Use this password to complete the setup wizard.

---

## Troubleshooting Steps

### Missing Jenkins Log File
If `/var/log/jenkins/jenkins.log` does not exist, create the required directories:
```bash
sudo mkdir -p /var/lib/jenkins /var/log/jenkins /var/cache/jenkins
sudo chown -R jenkins:jenkins /var/lib/jenkins /var/log/jenkins /var/cache/jenkins
```

### Port Conflicts
Ensure that port 8080 is not in use:
```bash
sudo netstat -tuln | grep 8080
```
Change the Jenkins port if necessary:
```bash
sudo nano /etc/default/jenkins
```
Modify the `HTTP_PORT` value and restart Jenkins:
```bash
sudo systemctl restart jenkins
```

### Manually Run Jenkins for Debugging
Run Jenkins in the foreground to identify issues:
```bash
sudo -u jenkins java -jar /usr/share/jenkins/jenkins.war
```

---

## Key Commands
- **Check Jenkins Status:**
  ```bash
  sudo systemctl status jenkins
  ```
- **View Logs:**
  ```bash
  sudo journalctl -xeu jenkins.service
  ```
- **Initial Admin Password:**
  ```bash
  sudo cat /var/lib/jenkins/secrets/initialAdminPassword
  ```

---

## Notes
- Ensure your security group allows traffic on port 8080.
- Regularly check Jenkins logs for any issues.

---

## Conclusion
This README provides a step-by-step guide to installing, configuring, and troubleshooting Jenkins on Ubuntu. Following these instructions ensures a successful setup and smooth operation of Jenkins on your EC2 instance.
