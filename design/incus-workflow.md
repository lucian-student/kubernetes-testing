# Incus workflow

1. distrobuilder builds image
2. import image with incus
3. for grouping vms its good idea to to create incus project?
4. then u need to make sure profile has disk device
5. features.networks,features.networks.zones

# Troubleshooting

Launching the instance
Error: Failed creating instance record: Failed initializing instance: Failed getting root disk: No root device could be found
* incus profile device add [profile] [name] disk pool=[pool name] path=[path]


Creating cp1
                                          
The instance you are starting doesn't have any network attached to it.
  To create a new network, use: incus network create
  To attach a network to an instance, use: incus network attach



fikus@DESKTOP-06RCDA2:~/projects/kubernetes-testing$ incus network create k8s
Error: Failed loading network: OVN isn't currently available