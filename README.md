# NAME

Net::Amazon::EC2 - Perl interface to the Amazon Elastic Compute Cloud (EC2)
environment.

# VERSION

EC2 Query API version: '2014-06-15'

# SYNOPSIS

    use Net::Amazon::EC2;

    my $ec2 = Net::Amazon::EC2->new(
           AWSAccessKeyId    => 'PUBLIC_KEY_HERE', 
           SecretAccessKey   => 'SECRET_KEY_HERE',
           signature_version => 4,
    );

    # Start 1 new instance from AMI: ami-XXXXXXXX
    my $instance = $ec2->run_instances(ImageId => 'ami-XXXXXXXX', MinCount => 1, MaxCount => 1);

    my $running_instances = $ec2->describe_instances;

    foreach my $reservation (@$running_instances) {
           foreach my $instance ($reservation->instances_set) {
                   print $instance->instance_id . "\n";
           }
    }

    my $instance_id = $instance->instances_set->[0]->instance_id;

    print "$instance_id\n";

    # Terminate instance

    my $result = $ec2->terminate_instances(InstanceId => $instance_id);

If an error occurs while communicating with EC2, these methods will 
throw a [Net::Amazon::EC2::Errors](https://metacpan.org/pod/Net::Amazon::EC2::Errors) exception.

# DESCRIPTION

This module is a Perl interface to Amazon's Elastic Compute Cloud. It uses the Query API to 
communicate with Amazon's Web Services framework.

# CLASS METHODS

## new(%params)

This is the constructor, it will return you a Net::Amazon::EC2 object to work with.  It takes 
these parameters:

- AWSAccessKeyId (required, unless an IAM role is present)

    Your AWS access key.  For information on IAM roles, see [http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/UsingIAM.html#UsingIAMrolesWithAmazonEC2Instances](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/UsingIAM.html#UsingIAMrolesWithAmazonEC2Instances)

- SecretAccessKey (required, unless an IAM role is present)

    Your secret key, **WARNING!** don't give this out or someone will be able to use your account 
    and incur charges on your behalf.

- SecurityToken (optional)

    When using temporary credentials from STS the Security Token must be passed
    in along with the temporary AWSAccessKeyId and SecretAccessKey.  The most common case is when using IAM credentials with the addition of MFA (multi-factor authentication).  See [http://docs.aws.amazon.com/STS/latest/UsingSTS/Welcome.html](http://docs.aws.amazon.com/STS/latest/UsingSTS/Welcome.html)

- region (optional)

    The region to run the API requests through. Defaults to us-east-1.

- ssl (optional)

    If set to a true value, the base\_url will use https:// instead of http://. Setting base\_url 
    explicitly will override this. Defaults to true as of 0.22.

- debug (optional)

    A flag to turn on debugging. Among other useful things, it will make the failing api calls print 
    a stack trace. It is turned off by default.

- return\_errors (optional)

    Previously, Net::Amazon::EC2 would return a [Net::Amazon::EC2::Errors](https://metacpan.org/pod/Net::Amazon::EC2::Errors) 
    object when it encountered an error condition. As of 0.19, this 
    object is thrown as an exception using croak or confess depending on
    if the debug flag is set.

    If you want/need the old behavior, set this attribute to a true value.

# OBJECT METHODS

## allocate\_address()

Acquires an elastic IP address which can be associated with an EC2-classic instance to create a movable static IP. Takes no arguments.

Returns the IP address obtained.

## allocate\_vpc\_address()

Acquires an elastic IP address which can be associated with a VPC instance to create a movable static IP. Takes no arguments.

Returns the allocationId of the allocated address.

## associate\_address(%params)

Associates an elastic IP address with an instance. It takes the following arguments:

- InstanceId (required)

    The instance id you wish to associate the IP address with

- PublicIp (optional)

    The IP address. Used for allocating addresses to EC2-classic instances.

- AllocationId (optional)

    The allocation ID.  Used for allocating address to VPC instances.

Returns true if the association succeeded.

## attach\_volume(%params)

Attach a volume to an instance.

- VolumeId (required)

    The volume id you wish to attach.

- InstanceId (required)

    The instance id you wish to attach the volume to.

- Device (required)

    The device id you want the volume attached as.

Returns a Net::Amazon::EC2::Attachment object containing the resulting volume status.

## authorize\_security\_group\_ingress(%params)

This method adds permissions to a security group.  It takes the following parameters:

- GroupName (required)

    The name of the group to add security rules to.

- SourceSecurityGroupName (required when authorizing a user and group together)

    Name of the group to add access for.

- SourceSecurityGroupOwnerId (required when authorizing a user and group together)

    Owner of the group to add access for.

- IpProtocol (required when adding access for a CIDR)

    IP Protocol of the rule you are adding access for (TCP, UDP, or ICMP)

- FromPort (required when adding access for a CIDR)

    Beginning of port range to add access for.

- ToPort (required when adding access for a CIDR)

    End of port range to add access for.

- CidrIp (required when adding access for a CIDR)

    The CIDR IP space we are adding access for.

Adding a rule can be done in two ways: adding a source group name + source group owner id, or, 
CIDR IP range. Both methods allow IP protocol, from port and to port specifications.

Returns 1 if rule is added successfully.

## bundle\_instance(%params)

Bundles the Windows instance. This procedure is not applicable for Linux and UNIX instances.

NOTE NOTE NOTE This is not well tested as I don't run windows instances

- InstanceId (required)

    The ID of the instance to bundle.

- Storage.S3.Bucket (required)

    The bucket in which to store the AMI. You can specify a bucket that you already own or a new bucket that Amazon EC2 creates on your behalf. If you specify a bucket that belongs to someone else, Amazon EC2 returns an error.

- Storage.S3.Prefix (required)

    Specifies the beginning of the file name of the AMI.

- Storage.S3.AWSAccessKeyId (required)

    The Access Key ID of the owner of the Amazon S3 bucket.

- Storage.S3.UploadPolicy (required)

    An Amazon S3 upload policy that gives Amazon EC2 permission to upload items into Amazon S3 on the user's behalf.

- Storage.S3.UploadPolicySignature (required)

    The signature of the Base64 encoded JSON document.

    JSON Parameters: (all are required)

    expiration - The expiration of the policy. Amazon recommends 12 hours or longer.
    conditions - A list of restrictions on what can be uploaded to Amazon S3. Must contain the bucket and ACL conditions in this table.
    bucket - The bucket to store the AMI. 
    acl - This must be set to ec2-bundle-read.

Returns a Net::Amazon::EC2::BundleInstanceResponse object

## cancel\_bundle\_task(%params)

Cancels the bundle task. This procedure is not applicable for Linux and UNIX instances.

- BundleId (required)

    The ID of the bundle task to cancel.

Returns a Net::Amazon::EC2::BundleInstanceResponse object

## confirm\_product\_instance(%params)

Checks to see if the product code passed in is attached to the instance id, taking the following parameter:

- ProductCode (required)

    The Product Code to check

- InstanceId (required)

    The Instance Id to check

Returns a Net::Amazon::EC2::ConfirmProductInstanceResponse object

## create\_image(%params)

Creates an AMI that uses an Amazon EBS root device from a "running" or "stopped" instance.

AMIs that use an Amazon EBS root device boot faster than AMIs that use instance stores. 
They can be up to 1 TiB in size, use storage that persists on instance failure, and can be stopped and started.

- InstanceId (required)

    The ID of the instance.

- Name (required)

    The name of the AMI that was provided during image creation.

    Note that the image name has the following constraints:

    3-128 alphanumeric characters, parenthesis, commas, slashes, dashes, or underscores.

- Description (optional)

    The description of the AMI that was provided during image creation.

- NoReboot (optional)

    By default this property is set to false, which means Amazon EC2 attempts to cleanly shut down the 
    instance before image creation and reboots the instance afterwards. When set to true, Amazon EC2 
    does not shut down the instance before creating the image. When this option is used, file system 
    integrity on the created image cannot be guaranteed. 

- BlockDeviceMapping (optional)

    Array ref of the device names exposed to the instance.

    You can specify device names as '&lt;device>=&lt;block\_device>' similar to ec2-create-image command. ([http://docs.aws.amazon.com/AWSEC2/latest/CommandLineReference/ApiReference-cmd-CreateImage.html](http://docs.aws.amazon.com/AWSEC2/latest/CommandLineReference/ApiReference-cmd-CreateImage.html))

        BlockDeviceMapping => [
              '/dev/sda=:256:true:standard',
              '/dev/sdb=none',
              '/dev/sdc=ephemeral0',
              '/dev/sdd=ephemeral1',
        ],

Returns the ID of the AMI created.

## create\_key\_pair(%params)

Creates a new 2048 bit key pair, taking the following parameter:

- KeyName (required)

    A name for this key. Should be unique.

Returns a Net::Amazon::EC2::KeyPair object

## create\_security\_group(%params)

This method creates a new security group.  It takes the following parameters:

- GroupName (required)

    The name of the new group to create.

- GroupDescription (required)

    A short description of the new group.

Returns 1 if the group creation succeeds.

## create\_snapshot(%params)

Create a snapshot of a volume. It takes the following arguments:

- VolumeId (required)

    The volume id of the volume you want to take a snapshot of.

- Description (optional)

    Description of the Amazon EBS snapshot.

Returns a Net::Amazon::EC2::Snapshot object of the newly created snapshot.

## create\_tags(%params)

Creates tags.

- ResourceId (required)

    The ID of the resource to create tags. Can be a scalar or arrayref

- Tags (required)

    Hashref where keys and values will be set on all resources given in the first element.

Returns true if the tag creation succeeded.

## create\_volume(%params)

Creates a volume.

- Size (required)

    The size in GiB ( 1024^3 ) of the volume you want to create.

- SnapshotId (optional)

    The optional snapshot id to create the volume from. The volume must
    be equal or larger than the snapshot it was created from.

- AvailabilityZone (required)

    The availability zone to create the volume in.

- VolumeType (optional)

    The volume type: 'standard', 'gp2', or 'io1'.  Defaults to 'standard'.

- Iops (required if VolumeType is 'io1')

    The number of I/O operations per second (IOPS) that the volume
    supports. This is limited to 30 times the volume size with an absolute maximum
    of 4000. It's likely these numbers will change in the future.

    Required when the volume type is io1; not used otherwise.

- Encrypted (optional)

    Encrypt the volume. EBS encrypted volumes are encrypted on the host using
    AWS managed keys. Only some instance types support encrypted volumes. At the
    time of writing encrypted volumes are not supported for boot volumes.

Returns a Net::Amazon::EC2::Volume object containing the resulting volume
status

## delete\_key\_pair(%params)

This method deletes a keypair.  Takes the following parameter:

- KeyName (required)

    The name of the key to delete.

Returns 1 if the key was successfully deleted.

## delete\_security\_group(%params)

This method deletes a security group.  It takes the following parameter:

- GroupName (required)

    The name of the security group to delete.

Returns 1 if the delete succeeded.

## delete\_snapshot(%params)

Deletes the snapshots passed in. It takes the following arguments:

- SnapshotId (required)

    A snapshot id can be passed in. Will delete the corresponding snapshot.

Returns true if the deleting succeeded.

## delete\_volume(%params)

Delete a volume.

- VolumeId (required)

    The volume id you wish to delete.

Returns true if the deleting succeeded.

## delete\_tags(%params)

Delete tags.

- ResourceId (required)

    The ID of the resource to delete tags

- Tag.Key (required)

    Key for a tag, may pass in a scalar or arrayref.

- Tag.Value (required)

    Value for a tag, may pass in a scalar or arrayref.

Returns true if the releasing succeeded.

## deregister\_image(%params)

This method will deregister an AMI. It takes the following parameter:

- ImageId (required)

    The image id of the AMI you want to deregister.

Returns 1 if the deregistering succeeded

## describe\_addresses(%params)

This method describes the elastic addresses currently allocated and any instances associated with them. It takes the following arguments:

- PublicIp (optional)

    The IP address to describe. Can be either a scalar or an array ref.

Returns an array ref of Net::Amazon::EC2::DescribeAddress objects

## describe\_availability\_zones(%params)

This method describes the availability zones currently available to choose from. It takes the following arguments:

- ZoneName (optional)

    The zone name to describe. Can be either a scalar or an array ref.

Returns an array ref of Net::Amazon::EC2::AvailabilityZone objects

## describe\_bundle\_tasks(%params)

Describes current bundling tasks. This procedure is not applicable for Linux and UNIX instances.

- BundleId (optional)

    The optional ID of the bundle task to describe.

Returns a array ref of Net::Amazon::EC2::BundleInstanceResponse objects

## describe\_image\_attributes(%params)

This method pulls a list of attributes for the image id specified

- ImageId (required)

    A scalar containing the image you want to get the list of attributes for.

- Attribute (required)

    A scalar containing the attribute to describe.

    Valid attributes are:

    - launchPermission - The AMIs launch permissions.
    - ImageId - ID of the AMI for which an attribute will be described.
    - productCodes - The product code attached to the AMI.
    - kernel - Describes the ID of the kernel associated with the AMI.
    - ramdisk - Describes the ID of RAM disk associated with the AMI.
    - blockDeviceMapping - Defines native device names to use when exposing virtual devices.
    - platform - Describes the operating system platform.

Returns a Net::Amazon::EC2::DescribeImageAttribute object

\* NOTE: There is currently a bug in Amazon's SOAP and Query API
for when you try and describe the attributes: kernel, ramdisk, blockDeviceMapping, or platform
AWS returns an invalid response. No response yet from Amazon on an ETA for getting that bug fixed.

## describe\_images(%params)

This method pulls a list of the AMIs which can be run.  The list can be modified by passing in some of the following parameters:

- ImageId (optional)

    Either a scalar or an array ref can be passed in, will cause just these AMIs to be 'described'

- Owner (optional)

    Either a scalar or an array ref can be passed in, will cause AMIs owned by the Owner's provided will be 'described'. Pass either account ids, or 'amazon' for all amazon-owned AMIs, or 'self' for your own AMIs.

- ExecutableBy (optional)

    Either a scalar or an array ref can be passed in, will cause AMIs executable by the account id's specified.  Or 'self' for your own AMIs.

Returns an array ref of Net::Amazon::EC2::DescribeImagesResponse objects

## describe\_instances(%params)

This method pulls a list of the instances which are running or were just running.  The list can be modified by passing in some of the following parameters:

- InstanceId (optional)

    Either a scalar or an array ref can be passed in, will cause just these instances to be 'described'

- Filter (optional)

    The filters for only the matching instances to be 'described'.
    A filter tuple is an arrayref constsing one key and one or more values.
    The option takes one filter tuple, or an arrayref of multiple filter tuples.

Returns an array ref of Net::Amazon::EC2::ReservationInfo objects

## describe\_instance\_status(%params)

This method pulls a list of the instances based on some status filter.  The list can be modified by passing in some of the following parameters:

- InstanceId (optional)

    Either a scalar or an array ref can be passed in, will cause just these instances to be 'described'

- Filter (optional)

    The filters for only the matching instances to be 'described'.
    A filter tuple is an arrayref constsing one key and one or more values.
    The option takes one filter tuple, or an arrayref of multiple filter tuples.

Returns an array ref of Net::Amazon::EC2::InstanceStatuses objects

## \_create\_describe\_instance\_status(%instanceElement)

Returns a blessed object. Used internally for wrapping describe\_instance\_status nextToken calls

- InstanceStatusElement (required)

    The instance status element we want to build out and return

Returns a Net::Amazon::EC2::InstanceStatuses object

## describe\_instance\_attribute(%params)

Returns information about an attribute of an instance. Only one attribute can be specified per call.

- InstanceId (required)

    The instance id we want to describe the attributes of.

- Attribute (required)

    The attribute we want to describe. Valid values are:

    - instanceType
    - kernel
    - ramdisk
    - userData
    - disableApiTermination
    - ebsOptimized
    - instanceInitiatedShutdownBehavior
    - rootDeviceName
    - sourceDestCheck
    - blockDeviceMapping

Returns a Net::Amazon::EC2::DescribeInstanceAttributeResponse object

## describe\_key\_pairs(%params)

This method describes the keypairs available on this account. It takes the following parameter:

- KeyName (optional)

    The name of the key to be described. Can be either a scalar or an array ref.

Returns an array ref of Net::Amazon::EC2::DescribeKeyPairsResponse objects

## describe\_regions(%params)

Describes EC2 regions that are currently available to launch instances in for this account.

- RegionName (optional)

    The name of the region(s) to be described. Can be either a scalar or an array ref.

Returns an array ref of Net::Amazon::EC2::Region objects

## describe\_reserved\_instances(%params)

Describes Reserved Instances that you purchased.

- ReservedInstancesId (optional)

    The reserved instance id(s) to be described. Can be either a scalar or an array ref.

Returns an array ref of Net::Amazon::EC2::ReservedInstance objects

## describe\_reserved\_instances\_offerings(%params)

Describes Reserved Instance offerings that are available for purchase. With Amazon EC2 Reserved Instances, 
you purchase the right to launch Amazon EC2 instances for a period of time (without getting insufficient 
capacity errors) and pay a lower usage rate for the actual time used.

- ReservedInstancesOfferingId (optional)

    ID of the Reserved Instances to describe.

- InstanceType (optional)

    The instance type. The default is m1.small. Amazon frequently updates their instance types.

    See http://aws.amazon.com/ec2/instance-types

- AvailabilityZone (optional)

    The Availability Zone in which the Reserved Instance can be used.

- ProductDescription (optional)

    The Reserved Instance description.

Returns an array ref of Net::Amazon::EC2::ReservedInstanceOffering objects

## describe\_security\_groups(%params)

This method describes the security groups available to this account. It takes the following parameter:

- GroupName (optional)

    The name of the security group(s) to be described. Can be either a scalar or an array ref.

- GroupId (optional)

    The id of the security group(s) to be described. Can be either a scalar or an array ref.

Returns an array ref of Net::Amazon::EC2::SecurityGroup objects

## describe\_snapshot\_attribute(%params)

Describes the snapshots attributes related to the snapshot in question. It takes the following arguments:

- SnapshotId (optional)

    Either a scalar or array ref of snapshot id's can be passed in. If this isn't passed in
    it will describe the attributes of all the current snapshots.

- Attribute (required)

    The attribute to describe, currently, the only valid attribute is createVolumePermission.

Returns a Net::Amazon::EC2::SnapshotAttribute object.

## describe\_snapshots(%params)

Describes the snapshots available to the user. It takes the following arguments:

- SnapshotId (optional)

    Either a scalar or array ref of snapshot id's can be passed in. If this isn't passed in
    it will describe all the current snapshots.

- Owner (optional)

    The owner of the snapshot.

- RestorableBy (optional)

    A user who can create volumes from the snapshot.

- Filter (optional)

    The filters for only the matching snapshots to be 'described'.  A
    filter tuple is an arrayref constsing one key and one or more values.
    The option takes one filter tuple, or an arrayref of multiple filter
    tuples.

Returns an array ref of Net::Amazon::EC2::Snapshot objects.

## describe\_volumes(%params)

Describes the volumes currently created. It takes the following arguments:

- VolumeId (optional)

    Either a scalar or array ref of volume id's can be passed in. If this isn't passed in
    it will describe all the current volumes.

Returns an array ref of Net::Amazon::EC2::Volume objects.

## describe\_subnets(%params)

This method describes the subnets on this account. It takes the following parameters:

- SubnetId (optional)

    The id of a subnet to be described.  Can either be a scalar or an array ref.

- Filter.Name (optional)

    The name of the Filter.Name to be described. Can be either a scalar or an array ref.
    See http://docs.aws.amazon.com/AWSEC2/latest/APIReference/ApiReference-query-DescribeSubnets.html
    for available filters.

- Filter.Value (optional)

    The name of the Filter.Value to be described. Can be either a scalar or an array ref.

Returns an array ref of Net::Amazon::EC2::DescribeSubnetResponse objects

## describe\_tags(%params)

This method describes the tags available on this account. It takes the following parameter:

- Filter.Name (optional)

    The name of the Filter.Name to be described. Can be either a scalar or an array ref.

- Filter.Value (optional)

    The name of the Filter.Value to be described. Can be either a scalar or an array ref.

Returns an array ref of Net::Amazon::EC2::DescribeTags objects

## detach\_volume(%params)

Detach a volume from an instance.

- VolumeId (required)

    The volume id you wish to detach.

- InstanceId (optional)

    The instance id you wish to detach from.

- Device (optional)

    The device the volume was attached as.

- Force (optional)

    A boolean for if to forcibly detach the volume from the instance.
    WARNING: This can lead to data loss or a corrupted file system.
    	   Use this option only as a last resort to detach a volume
    	   from a failed instance.  The instance will not have an
    	   opportunity to flush file system caches nor file system
    	   meta data.

Returns a Net::Amazon::EC2::Attachment object containing the resulting volume status.

## disassociate\_address(%params)

Disassociates an elastic IP address with an instance. It takes the following arguments:

- PublicIp (conditional)

    The IP address to disassociate, mandatory to remove an IP from a EC2-classic instance.

- AssociationId (conditional)

    The Association ID of an IP address, mandatory to remove an IP from a VPC instance.

Returns true if the disassociation succeeded.

## get\_console\_output(%params)

This method gets the output from the virtual console for an instance.  It takes the following parameters:

- InstanceId (required)

    A scalar containing a instance id.

Returns a Net::Amazon::EC2::ConsoleOutput object or `undef` if there is no
new output. (This can happen in cases where the console output has not changed
since the last call.)

## get\_password\_data(%params)

Retrieves the encrypted administrator password for the instances running Windows. This procedure is not applicable for Linux and UNIX instances.

- InstanceId (required)

    The Instance Id for which to retrieve the password.

Returns a Net::Amazon::EC2::InstancePassword object

## modify\_image\_attribute(%params)

This method modifies attributes of an machine image.

- ImageId (required)

    The AMI to modify the attributes of.

- Attribute (required)

    The attribute you wish to modify, right now the attributes you can modify are launchPermission and productCodes

- OperationType (required for launchPermission)

    The operation you wish to perform on the attribute. Right now just 'add' and 'remove' are supported.

- UserId (required for launchPermission)

    User Id's you wish to add/remove from the attribute.

- UserGroup (required for launchPermission)

    Groups you wish to add/remove from the attribute.  Currently there is only one User Group available 'all' for all Amazon EC2 customers.

- ProductCode (required for productCodes)

    Attaches a product code to the AMI. Currently only one product code can be assigned to the AMI.  Once this is set it cannot be changed or reset.

Returns 1 if the modification succeeds.

## modify\_instance\_attribute(%params)

Modify an attribute of an instance. 

- InstanceId (required)

    The instance id we want to modify the attributes of.

- Attribute (required)

    The attribute we want to modify. Valid values are:

    - instanceType
    - kernel
    - ramdisk
    - userData
    - disableApiTermination
    - instanceInitiatedShutdownBehavior
    - rootDeviceName
    - blockDeviceMapping

- Value (required)

    The value to set the attribute to.

    You may also pass a hashref with one or more keys 
    and values. This hashref will be flattened and 
    passed to AWS.

    For example:

            $ec2->modify_instance_attribute(
                    'InstanceId' => $id,
                    'Attribute' => 'blockDeviceMapping',
                    'Value' => {
                            'BlockDeviceMapping.1.DeviceName' => '/dev/sdf1',
                            'BlockDeviceMapping.1.Ebs.DeleteOnTermination' => 'true',
                    }
            );

Returns 1 if the modification succeeds.

## modify\_snapshot\_attribute(%params)

This method modifies attributes of a snapshot.

- SnapshotId (required)

    The snapshot id to modify the attributes of.

- UserId (optional)

    User Id you wish to add/remove create volume permissions for.

- UserGroup (optional)

    User Id you wish to add/remove create volume permissions for. To make the snapshot createable by all
    set the UserGroup to "all".

- Attribute (required)

    The attribute you wish to modify, right now the only attribute you can modify is "CreateVolumePermission" 

- OperationType (required)

    The operation you wish to perform on the attribute. Right now just 'add' and 'remove' are supported.

Returns 1 if the modification succeeds.

## monitor\_instances(%params)

Enables monitoring for a running instance. For more information, refer to the Amazon CloudWatch Developer Guide.

- InstanceId (required)

    The instance id(s) to monitor. Can be a scalar or an array ref

Returns an array ref of Net::Amazon::EC2::MonitoredInstance objects

## purchase\_reserved\_instances\_offering(%params)

Purchases a Reserved Instance for use with your account. With Amazon EC2 Reserved Instances, you purchase the right to 
launch Amazon EC2 instances for a period of time (without getting insufficient capacity errors) and pay a lower usage 
rate for the actual time used.

- ReservedInstancesOfferingId (required)

    ID of the Reserved Instances to describe. Can be either a scalar or an array ref.

- InstanceCount (optional)

    The number of Reserved Instances to purchase (default is 1). Can be either a scalar or an array ref.

    NOTE NOTE NOTE, the array ref needs to line up with the InstanceCount if you want to pass that in, so that 
    the right number of instances are started of the right instance offering

Returns 1 if the reservations succeeded.

## reboot\_instances(%params)

This method reboots an instance.  It takes the following parameters:

- InstanceId (required)

    Instance Id of the instance you wish to reboot. Can be either a scalar or array ref of instances to reboot.

Returns 1 if the reboot succeeded.

## register\_image(%params)

This method registers an AMI on the EC2. It takes the following parameter:

- ImageLocation (optional)

    The location of the AMI manifest on S3

- Name (required)

    The name of the AMI that was provided during image creation.

- Description (optional)

    The description of the AMI.

- Architecture (optional)

    The architecture of the image. Either i386 or x86\_64

- KernelId (optional)

    The ID of the kernel to select. 

- RamdiskId (optional)

    The ID of the RAM disk to select. Some kernels require additional drivers at launch. 

- RootDeviceName (optional)

    The root device name (e.g., /dev/sda1).

- BlockDeviceMapping (optional)

    This needs to be a data structure like this:

        [
               {
                       deviceName      => "/dev/sdh", (optional)
                       virtualName     => "ephemerel0", (optional)
                       noDevice        => "/dev/sdl", (optional),
                       ebs                     => {
                               snapshotId                      => "snap-0000", (optional)
                               volumeSize                      => "20", (optional)
                               deleteOnTermination     => "false", (optional)
                       },
               },
               ...
        ]      

Returns the image id of the new image on EC2.

## release\_address(%params)

Releases an allocated IP address. It takes the following arguments:

- PublicIp (required)

    The IP address to release

Returns true if the releasing succeeded.

## reset\_image\_attribute(%params)

This method resets an attribute for an AMI to its default state (NOTE: product codes cannot be reset).  
It takes the following parameters:

- ImageId (required)

    The image id of the AMI you wish to reset the attributes on.

- Attribute (required)

    The attribute you want to reset.

Returns 1 if the attribute reset succeeds.

## reset\_instance\_attribute(%params)

Reset an attribute of an instance. Only one attribute can be specified per call.

- InstanceId (required)

    The instance id we want to reset the attributes of.

- Attribute (required)

    The attribute we want to reset. Valid values are:

    - kernel
    - ramdisk

Returns 1 if the reset succeeds.

## reset\_snapshot\_attribute(%params)

This method resets an attribute for an snapshot to its default state.

It takes the following parameters:

- SnapshotId (required)

    The snapshot id of the snapshot you wish to reset the attributes on.

- Attribute (required)

    The attribute you want to reset (currently "CreateVolumePermission" is the only
    valid attribute).

Returns 1 if the attribute reset succeeds.

## revoke\_security\_group\_ingress(%params)

This method revoke permissions to a security group.  It takes the following parameters:

- GroupName (required)

    The name of the group to revoke security rules from.

- SourceSecurityGroupName (required when revoking a user and group together)

    Name of the group to revoke access from.

- SourceSecurityGroupOwnerId (required when revoking a user and group together)

    Owner of the group to revoke access from.

- IpProtocol (required when revoking access from a CIDR)

    IP Protocol of the rule you are revoking access from (TCP, UDP, or ICMP)

- FromPort (required when revoking access from a CIDR)

    Beginning of port range to revoke access from.

- ToPort (required when revoking access from a CIDR)

    End of port range to revoke access from.

- CidrIp (required when revoking access from a CIDR)

    The CIDR IP space we are revoking access from.

Revoking a rule can be done in two ways: revoking a source group name + source group owner id, or, by Protocol + start port + end port + CIDR IP.  The two are mutally exclusive.

Returns 1 if rule is revoked successfully.

## run\_instances(%params)

This method will start instance(s) of AMIs on EC2. The parameters
indicate which AMI to instantiate and how many / what properties they
have:

- ImageId (required)

    The image id you want to start an instance of.

- MinCount (required)

    The minimum number of instances to start.

- MaxCount (required)

    The maximum number of instances to start.

- KeyName (optional)

    The keypair name to associate this instance with.  If omitted, will use your default keypair.

- SecurityGroup (optional)

    An scalar or array ref. Will associate this instance with the group names passed in.  If omitted, will be associated with the default security group.

- SecurityGroupId (optional)

    An scalar or array ref. Will associate this instance with the group ids passed in.  If omitted, will be associated with the default security group.

- AdditionalInfo (optional)

    Specifies additional information to make available to the instance(s).

- UserData (optional)

    Optional data to pass into the instance being started.  Needs to be base64 encoded.

- InstanceType (optional)

    Specifies the type of instance to start.

    See http://aws.amazon.com/ec2/instance-types

    The options are:

    - m1.small (default)

        1 EC2 Compute Unit (1 virtual core with 1 EC2 Compute Unit). 32-bit or 64-bit, 1.7GB RAM, 160GB disk

    - m1.medium Medium Instance

        2 EC2 Compute Units (1 virtual core with 2 EC2 Compute Unit), 32-bit or 64-bit, 3.75GB RAM, 410GB disk

    - m1.large: Standard Large Instance

        4 EC2 Compute Units (2 virtual cores with 2 EC2 Compute Units each). 64-bit, 7.5GB RAM, 850GB disk

    - m1.xlarge: Standard Extra Large Instance

        8 EC2 Compute Units (4 virtual cores with 2 EC2 Compute Units each). 64-bit, 15GB RAM, 1690GB disk

    - t1.micro Micro Instance

        Up to 2 EC2 Compute Units (for short periodic bursts), 32-bit or 64-bit, 613MB RAM, EBS storage only

    - c1.medium: High-CPU Medium Instance

        5 EC2 Compute Units (2 virtual cores with 2.5 EC2 Compute Units each). 32-bit or 64-bit, 1.7GB RAM, 350GB disk

    - c1.xlarge: High-CPU Extra Large Instance

        20 EC2 Compute Units (8 virtual cores with 2.5 EC2 Compute Units each). 64-bit, 7GB RAM, 1690GB disk

    - m2.2xlarge High-Memory Double Extra Large Instance

        13 EC2 Compute Units (4 virtual cores with 3.25 EC2 Compute Units each). 64-bit, 34.2GB RAM, 850GB disk

    - m2.4xlarge High-Memory Quadruple Extra Large Instance

        26 EC2 Compute Units (8 virtual cores with 3.25 EC2 Compute Units each). 64-bit, 68.4GB RAM, 1690GB disk

    - cc1.4xlarge Cluster Compute Quadruple Extra Large Instance

        33.5 EC2 Compute Units (2 x Intel Xeon X5570, quad-core "Nehalem" architecture), 64-bit, 23GB RAM, 1690GB disk, 10Gbit Ethernet

    - cc1.8xlarge Cluster Compute Eight Extra Large Instance

        88 EC2 Compute Units (2 x Intel Xeon E5-2670, eight-core "Sandy Bridge" architecture), 64-bit, 60.5GB RAM, 3370GB disk, 10Gbit Ethernet

    - cg1.4xlarge Cluster GPU Quadruple Extra Large Instance

        33.5 EC2 Compute Units (2 x Intel Xeon X5570, quad-core "Nehalem" architecture), 64-bit, 22GB RAM 1690GB disk, 10Gbit Ethernet, 2 x NVIDIA Tesla "Fermi" M2050 GPUs

    - hi1.4xlarge High I/O Quadruple Extra Large Instance

        35 EC2 Compute Units (16 virtual cores), 60.5GB RAM, 64-bit, 2 x 1024GB SSD disk, 10Gbit Ethernet

- Placement.AvailabilityZone (optional)

    The availability zone you want to run the instance in

- KernelId (optional)

    The id of the kernel you want to launch the instance with

- RamdiskId (optional)

    The id of the ramdisk you want to launch the instance with

- BlockDeviceMapping.VirtualName (optional)

    This is the virtual name for a blocked device to be attached, may pass in a scalar or arrayref

- BlockDeviceMapping.DeviceName (optional)

    This is the device name for a block device to be attached, may pass in a scalar or arrayref

- BlockDeviceMapping.Ebs.VolumeSize (optional)

    Specifies the size of the root block device as an integer (GB). If used, required
    that `BlockDeviceMapping.DeviceName` also be specified. One may pass in a
    scalar or arrayref. This parameter will override the disk size settings implied
    by the `InstanceType`, if used.

    Additional volume related parameters include,

    - BlockDeviceMapping.Ebs.SnapshotId

        May pass in a scalar or arrayref.

    - BlockDeviceMapping.Ebs.VolumeType

        May pass in a scalar or arrayref.

    - BlockDeviceMapping.Ebs.DeleteOnTermination

        May pass in a scalar or arrayref.

- Encoding (optional)

    The encoding.

- Version (optional)

    The version.

- Monitoring.Enabled (optional)

    Enables monitoring for this instance.

- SubnetId (optional)

    Specifies the subnet ID within which to launch the instance(s) for Amazon Virtual Private Cloud.

- ClientToken (optional)

    Specifies the idempotent instance id.

- EbsOptimized (optional)

    Whether the instance is optimized for EBS I/O.

- PrivateIpAddress (optional)

    Specifies the private IP address to use when launching an Amazon VPC instance.

- IamInstanceProfile.Name (optional)

    Specifies the IAM profile to associate with the launched instance(s).  This is the name of the role.

- IamInstanceProfile.Arn (optional)

    Specifies the IAM profile to associate with the launched instance(s).  This is the ARN of the profile.

Returns a Net::Amazon::EC2::ReservationInfo object

## start\_instances(%params)

Starts an instance that uses an Amazon EBS volume as its root device.

- InstanceId (required)

    Either a scalar or an array ref can be passed in (containing instance ids to be started).

Returns an array ref of Net::Amazon::EC2::InstanceStateChange objects.

## stop\_instances(%params)

Stops an instance that uses an Amazon EBS volume as its root device.

- InstanceId (required)

    Either a scalar or an array ref can be passed in (containing instance ids to be stopped).

- Force (optional)

    If set to true, forces the instance to stop. The instance will not have an opportunity to 
    flush file system caches nor file system meta data. If you use this option, you must perform file 
    system check and repair procedures. This option is not recommended for Windows instances.

    The default is false.

Returns an array ref of Net::Amazon::EC2::InstanceStateChange objects.

## terminate\_instances(%params)

This method shuts down instance(s) passed into it. It takes the following parameter:

- InstanceId (required)

    Either a scalar or an array ref can be passed in (containing instance ids)

Returns an array ref of Net::Amazon::EC2::InstanceStateChange objects.

## unmonitor\_instances(%params)

Disables monitoring for a running instance. For more information, refer to the Amazon CloudWatch Developer Guide.

- InstanceId (required)

    The instance id(s) to monitor. Can be a scalar or an array ref

Returns an array ref of Net::Amazon::EC2::MonitoredInstance objects

# TESTING

Set AWS\_ACCESS\_KEY\_ID and SECRET\_ACCESS\_KEY environment variables to run the live tests.  
Note: because the live tests start an instance (and kill it) in both the tests and backwards compat tests there will be 2 hours of 
machine instance usage charges (since there are 2 instances started) which as of February 1st, 2010 costs a total of $0.17 USD

Important note about the windows-only methods.  These have not been well tested as I do not run windows-based instances, so exercise
caution in using these.

# BUGS

Please report any bugs or feature requests to `bug-net-amazon-ec2 at rt.cpan.org`, or through
the web interface at [http://rt.cpan.org/NoAuth/ReportBug.html?Queue=Net-Amazon-EC2](http://rt.cpan.org/NoAuth/ReportBug.html?Queue=Net-Amazon-EC2).  I will 
be notified, and then you'll automatically be notified of progress on your bug as I make changes.

# AUTHOR

Jeff Kim <cpan@chosec.com>

# CONTRIBUTORS

John McCullough and others as listed in the Changelog

# MAINTAINER

The current maintainer is Mark Allen `<mallen@cpan.org>`

# COPYRIGHT

Copyright (c) 2006-2010 Jeff Kim. 

Copyright (c) 2012 Mark Allen.

This program is free software; you can redistribute it and/or modify it
under the same terms as Perl itself.

# SEE ALSO

Amazon EC2 API: [http://docs.amazonwebservices.com/AWSEC2/latest/APIReference/](http://docs.amazonwebservices.com/AWSEC2/latest/APIReference/)
