# 📦 Module 6 Demo Project
# Run Nexus on DigitalOcean & Publish Java Artifacts

## 🛠️ Technologies Used
- **Nexus Repository Manager** – Artifact repository for Java builds.
- **DigitalOcean** – Cloud platform for hosting Nexus.
- **Ubuntu Linux** – OS for the cloud server.
- **Java** – Runtime for building backend applications.
- **Gradle & Maven** – Build tools used to compile and publish artifacts.

---

## 📄 Project Description

This project demonstrates how to install and configure **Sonatype Nexus** from scratch on a cloud server, and how to upload Java build artifacts to it using both **Gradle** and **Maven**.

---

## ✅ Step-by-Step Guide

### Step 1: Provision a Droplet on DigitalOcean
- Use Ubuntu 22.04 LTS
- Enable SSH access
- Allocate sufficient RAM (2 GB minimum recommended for Nexus)

---

### Step 2: Install and Start Nexus Repository Manager
```bash
# Update and install dependencies
sudo apt update && sudo apt install -y openjdk-8-jdk wget

# Add a new user for Nexus
sudo adduser nexus
sudo usermod -aG sudo nexus

# Switch to nexus user
su - nexus

# Download and extract Nexus
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -xvzf latest-unix.tar.gz
mv nexus-3* nexus
mv sonatype-work/ nexus-data

# Edit startup script
cd nexus/bin
nano nexus.rc
# Add:
run_as_user="nexus"

# Start Nexus
./nexus start

# Enable Nexus to run on boot (optional)
sudo ln -s /home/nexus/nexus/bin/nexus /etc/init.d/nexus
sudo systemctl enable nexus
```

Access Nexus at:
```
http://<DROPLET_PUBLIC_IP>:8081
```

---

### Step 3: Configure Nexus
- Log in using default credentials: `admin` / (found in `/nexus-data/admin.password`)
- Change admin password
- Create a **new hosted repository** for Maven/Gradle (e.g., `maven-releases`)
- Create a **new user** with permissions to deploy to that repository

---

### Step 4: Publish Artifact – Java Gradle Project

In `build.gradle`:
```groovy
publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
    repositories {
        maven {
            name = "nexus"
            url = "http://<DROPLET_PUBLIC_IP>:8081/repository/maven-releases/"
            credentials {
                username = "your-nexus-username"
                password = "your-nexus-password"
            }
        }
    }
}
```

Then run:
```bash
gradle publish
```

---

### Step 5: Publish Artifact – Java Maven Project

In `pom.xml`, add:
```xml
<distributionManagement>
    <repository>
        <id>nexus</id>
        <url>http://<DROPLET_PUBLIC_IP>:8081/repository/maven-releases/</url>
    </repository>
</distributionManagement>
```

Add credentials to `~/.m2/settings.xml`:
```xml
<servers>
  <server>
    <id>nexus</id>
    <username>your-nexus-username</username>
    <password>your-nexus-password</password>
  </server>
</servers>
```

Then run:
```bash
mvn deploy
```

---

## 🎯 Result

Both **Gradle** and **Maven** Java projects are successfully built and deployed to your self-hosted **Nexus Repository** on DigitalOcean.

You now have a fully functional private artifact repository system.
