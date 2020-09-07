# Lab with Vagrant
We will use Hyper-V as provider.

1. Install / Enable Hyper-V 
2. Install Vagrant
3. Clone this repository
4. On your machine, create a local "vagrant" user for the SMB share and assign the user to the directory created at the step before
5. Start a PowerShell console as "Administrator"
7. Change the local directory :
```
cd c:\yourpath\GettingStarted
```
6. Start the machine
```
vagrant up --provider=hyperv
```

7. If prompted, chose the correct switch
```
    default: Please choose a switch to attach to your Hyper-V instance.
    default: If none of these are appropriate, please open the Hyper-V manager
    default: to create a new virtual switch.
    default:
    default: 1) Default Switch
    default: 2) WSL
    default:
    default: What switch would you like to use? 1
```

8. When prompted, enter the SMB credentials created before:
```
    default:
    default: Username (user[@domain]): vagrant
    default: Password (will be hidden):
```

9. Log in the machine
```
vagrant ssh
```

10. Stop the machine
```
vagrant suspend
```

11. Restart the machine
```
vagrant resume
```

12. Destroy the machine
```
vagrant destroy
```