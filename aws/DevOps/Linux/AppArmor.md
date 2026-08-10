[[Linux]]
[[DevOps/Kubernetes/Security/Security]]
[[DevSecOps/Security|Security]]

## AppArmor Vs [[SELinux]]

AppArmor and SELinux both protect systems from malicious software and unauthorized access, but they work differently.

|Aspect|AppArmor|SELinux|
|---|---|---|
|Approach|Uses profile files for each program to allow or block actions|Uses centralized policies that define exact permissions for users, programs, and resources|
|Distribution|Mainly used on Ubuntu and SUSE Linux|Primarily used on RHEL, CentOS, and Fedora|
|Complexity|Easier to set up and manage, but has less granular control|More complex but provides very detailed control over permissions|
|Label System|Path-based access control|Label-based (context) access control|
https://www.geeksforgeeks.org/linux-unix/what-is-selinux/
