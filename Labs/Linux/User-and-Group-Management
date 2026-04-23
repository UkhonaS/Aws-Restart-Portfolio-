# 🐧 Linux User & Group Management — AWS EC2 Lab

This lab had me working directly on an Amazon Linux EC2 instance via SSH, tackling one of the most fundamental sysadmin tasks: **setting up users, organizing them into groups, and seeing firsthand what happens when someone tries to do something they don't have permission to do.**

---

## 🎯 What I Practiced

- Creating Linux user accounts and assigning passwords
- Creating groups and placing users into them based on job roles
- Understanding how file permissions work in practice
- Learning what the `sudoers` file is and why it matters
- Seeing how Linux logs unauthorized privilege escalation attempts

---

## 👥 Users I Created

| User ID | Name | Job Role |
|---|---|---|
| `arosalez` | Alejandro Rosalez | Sales Manager |
| `eowusu` | Efua Owusu | Shipping |
| `jdoe` | Jane Doe | Shipping |
| `ljuan` | Li Juan | HR Manager |
| `mmajor` | Mary Major | Finance Manager |
| `mjackson` | Mateo Jackson | CEO |
| `nwolf` | Nikki Wolf | Sales Representative |
| `psantos` | Paulo Santos | Shipping |
| `smartinez` | Sofia Martinez | HR Specialist |
| `ssarkar` | Saanvi Sarkar | Finance Specialist |

All users were created with the default starting password `P@ssword1234!`

---

## 🗂️ Groups & Membership

| Group | Members |
|---|---|
| Sales | arosalez, nwolf |
| HR | ljuan, smartinez |
| Finance | mmajor, ssarkar |
| Shipping | eowusu, jdoe, psantos |
| Managers | arosalez, ljuan, mmajor |
| CEO | mjackson |

> I also added `ec2-user` to all groups for administrative access.

One thing that stood out here: **managers also belong to the Personnel group, but not every person in Personnel is a manager.** Some users intentionally sit in multiple groups — which is a realistic reflection of how org structures actually work.

---

## 🔧 Commands I Used

````bash
# Create a new user
sudo useradd arosalez

# Set their password
sudo passwd arosalez

# Verify users were created
sudo cat /etc/passwd | cut -d: -f1

# Create a group
sudo groupadd Sales

# Add a user to a group
sudo usermod -a -G Sales arosalez

# Verify group memberships
cat /etc/group
````

---

## 🔐 The Sudoers Lesson

After setting everything up, I switched into `arosalez`'s account to test what a regular user can and can't do:

````bash
su arosalez

# Attempting to create a file in ec2-user's home directory
touch myFile.txt
# → touch: cannot touch 'myFile.txt': Permission denied

# Trying to force it with sudo
sudo touch myFile.txt
# → arosalez is not in the sudoers file. This incident will be reported.
````

This was a solid real-world demonstration of **least privilege** — users only get access to what they need, nothing beyond that. What made it click even more was checking the logs afterward:

````bash
sudo cat /var/log/secure
# → arosalez : user NOT in sudoers ; TTY=pts/1 ; PWD=/home/ec2-user ; USER=root ; COMMAND=/bin/touch myFile.txt
````

Linux doesn't just block the action — it records it. In any real environment, that log entry would be flagged during a security audit.

---

## 💡 Key Takeaways

- User and group management is the foundation of access control on any Linux system
- `/etc/passwd` and `/etc/group` are where all user and group data is stored
- `sudo` is a privilege that has to be explicitly granted — it's not something users just have
- Linux keeps receipts — failed privilege escalation attempts are always logged

---

## ⚙️ Environment

- **Platform:** AWS EC2
- **OS:** Amazon Linux
- **Access:** SSH with key-pair authentication

````
````
