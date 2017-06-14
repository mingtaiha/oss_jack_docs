
First Set of Objectives

- Make it a Fedora or CentOS 7 machine
  - RHEL is an industry standard, both are a free versions of it
    - feel free to dual boot the spare machine if necessary
    - NFS server
    - Make sure this service comes up after the machine reboots
    - Make the NFS partition available to one client machine on your subnet
    - Make sure only one group of users has access to that (ie: even if it's mounted on your client machine, some users won't be able to get to it)
    - Keep a log of what you've done and the steps you took.
      - Be able to tell me all about it.
      - Remember, this is just the beginning


Second Set of Objectives

find the difference between a hard mount vs. a soft mount

get fstab working on your laptop
- in the event the partition is available, it will mount
- if not, make sure it doesn't try to mount it

see if UUID's are set up for your partition
- make sure that your partition will mount for your machine and yours alone
  - use authentication or other means as needed

  split off the partition
  - (ie: make the shared NFS partition separate from the root filesystem)
  - for extra points, see if you can do this without reinstalling the machine


  takeaway:
  - a service isn't production until:
    - it can be reinstalled quickly
      - it's backed up
        - it's monitored
