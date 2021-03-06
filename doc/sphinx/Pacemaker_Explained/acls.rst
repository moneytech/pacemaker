.. _acl:

Access Control Lists (ACLs)
---------------------------

.. Convert_to_RST:

   anchor:ch-acls[Chapter 13, ACLs]
   indexterm:[access control list]
   indexterm:[ACL]
   
   By default, the +root+ user or any user in the +haclient+ group can modify
   Pacemaker's CIB without restriction. Pacemaker offers 'access control lists
   (ACLs)' to provide more fine-grained authorization.
   
   == ACL Prerequisites ==
   
   In order to use ACLs:
   
   * The Pacemaker software must have been compiled with ACL support. If the
     output of the command `pacemakerd --features` contains `acls`, your
     installation supports ACLs.
   
   * Desired users must have user accounts in the +haclient+ group on all nodes in
     the cluster.
   
   * If your CIB was created before Pacemaker 1.1.12, it may need to be updated to
     the current schema using `cibadmin --upgrade` in order to use the syntax
     documented here.
   
   * The +enable-acl+ <<s-cluster-options,cluster option>> must be set to true.
   
   == ACL Configuration ==
   
   ACLs are specified within an +acls+ element of the CIB. The +acls+ element may
   contain any number of +acl_role+, +acl_target+, and +acl_group+ elements.
   
   == ACL Roles ==
   
   An ACL role is a collection of permissions allowing or denying access to
   particular portions of the CIB.
   
   .Properties of an ACL Role
   [width="95%",cols="1m,<3",options="header",align="center"]
   |====
   
   |Attribute
   |Description
   
   |id
   |A unique name for the role (required)
    indexterm:[id,acl_role]
    indexterm:[access control list,acl_role,id]
   
   |description
   |Arbitrary text (not used by Pacemaker)
    indexterm:[description,acl_role]
    indexterm:[access control list,acl_role,description]
   
   |====
   
   An +acl_role+ element may contain any number of +acl_permission+ elements.
   
   .Properties of an ACL Permission
   [width="95%",cols="1m,<3",options="header",align="center"]
   |====
   
   |Attribute
   |Description
   
   |id
   |A unique name for the permission (required)
    indexterm:[id,acl_permission]
    indexterm:[access control list,acl_permission,id]
   
   |description
   |Arbitrary text (not used by Pacemaker)
    indexterm:[description,acl_permission]
    indexterm:[access control list,acl_permission,description]
   
   |kind
   |The access being granted. Allowed values are +read+, +write+, and +deny+.
    A value of +write+ grants both read and write access.
    indexterm:[kind,acl_permission]
    indexterm:[access control list,acl_permission,kind]
   
   |object-type
   |The name of an XML element in the CIB to which the permission applies.
    (Exactly one of +object-type+, +xpath+, and +reference+ must be specified for
    a permission.)
    indexterm:[object-type,acl_permission]
    indexterm:[access control list,acl_permission,object-type]
   
   |attribute
   |If specified, the permission applies only to +object-type+ elements that have
    this attribute set (to any value). If not specified, the permission applies to
    all +object-type+ elements. May only be used with +object-type+.
    indexterm:[attribute,acl_permission]
    indexterm:[access control list,acl_permission,attribute]
   
   |reference
   |The ID of an XML element in the CIB to which the permission applies.
    (Exactly one of +object-type+, +xpath+, and +reference+ must be specified for
    a permission.)
    indexterm:[reference,acl_permission]
    indexterm:[access control list,acl_permission,reference]
   
   |xpath
   |An https://www.w3.org/TR/xpath-10/[XPath] specification selecting an XML
    element in the CIB to which the permission applies. Attributes may be
    specified in the XPath to select particular elements, but the permissions
    apply to the entire element.
    (Exactly one of +object-type+, +xpath+, and +reference+ must be specified for
    a permission.)
    indexterm:[xpath,acl_permission]
    indexterm:[access control list,acl_permission,xpath]
   
   |====
   
   [IMPORTANT]
   ====
   * Permissions are applied to the selected XML element's entire XML subtree
     (all elements enclosed within it).
   
   * Write permission grants the ability to create, modify, or remove the element
     and its subtree, and also the ability to create any "scaffolding" elements
     (enclosing elements that do not have attributes other than an ID).
   
   * Permissions for more specific matches (more deeply nested elements) take
     precedence over more general ones.
   
   * If multiple permissions are configured for the same match (for example, in
     different roles applied to the same user), any +deny+ permission takes
     precedence, then +write+, then lastly +read+.
   ====
   
   == ACL Targets and Groups ==
   
   ACL targets correspond to user accounts on the system.
   
   .Properties of an ACL Target
   [width="95%",cols="1m,<3",options="header",align="center"]
   |====
   
   |Attribute
   |Description
   
   |id
   |The name of a user on the system (required)
    indexterm:[id,acl_target]
    indexterm:[access control list,acl_target,id]
   
   |====
   
   ACL groups may be specified, but are not currently used by Pacemaker. This is
   expected to change in a future version.
   
   .Properties of an ACL Group
   [width="95%",cols="1m,<3",options="header",align="center"]
   |====
   
   |Attribute
   |Description
   
   |id
   |The name of a group on the system (required)
    indexterm:[id,acl_group]
    indexterm:[access control list,acl_group,id]
   
   |====
   
   Each +acl_target+ and +acl_group+ element may contain any number of +role+
   elements.
   
   .Properties of an ACL Role Reference
   [width="95%",cols="1m,<3",options="header",align="center"]
   |====
   
   |Attribute
   |Description
   
   |id
   |The +id+ of an +acl_role+ element that specifies permissions granted to the
    enclosing target or group
    indexterm:[id,role]
    indexterm:[access control list,role,id]
   
   |====
   
   [IMPORTANT]
   ====
   The +root+ and +hacluster+ user accounts always have full access to the CIB,
   regardless of ACLs. For other user accounts, when +enable-acl+ is true,
   permission to all parts of the CIB is denied by default (permissions must be
   explicitly granted).
   ====
   
   == ACL Examples ==
   
   [source,XML]
   ----
   <acls>
   
      <acl_role id="read_all">
          <acl_permission id="read_all-cib" kind="read" xpath="/cib" />
      </acl_role>
   
      <acl_role id="operator">
   
          <acl_permission id="operator-maintenance-mode" kind="write"
              xpath="//crm_config//nvpair[@name='maintenance-mode']" />
   
          <acl_permission id="operator-maintenance-attr" kind="write"
              xpath="//nvpair[@name='maintenance']" />
   
          <acl_permission id="operator-target-role" kind="write"
              xpath="//resources//meta_attributes/nvpair[@name='target-role']" />
   
          <acl_permission id="operator-is-managed" kind="write"
              xpath="//resources//nvpair[@name='is-managed']" />
   
          <acl_permission id="operator-rsc_location" kind="write"
              object-type="rsc_location" />
   
      </acl_role>
   
      <acl_role id="administrator">
          <acl_permission id="administrator-cib" kind="write" xpath="/cib" />
      </acl_role>
   
      <acl_role id="minimal">
   
          <acl_permission id="minimal-standby" kind="read"
              description="allow reading standby node attribute (permanent or transient)"
              xpath="//instance_attributes/nvpair[@name='standby']"/>
   
          <acl_permission id="minimal-maintenance" kind="read"
              description="allow reading maintenance node attribute (permanent or transient)"
              xpath="//nvpair[@name='maintenance']"/>
   
          <acl_permission id="minimal-target-role" kind="read"
              description="allow reading resource target roles"
              xpath="//resources//meta_attributes/nvpair[@name='target-role']"/>
   
          <acl_permission id="minimal-is-managed" kind="read"
              description="allow reading resource managed status"
              xpath="//resources//meta_attributes/nvpair[@name='is-managed']"/>
   
          <acl_permission id="minimal-deny-instance-attributes" kind="deny"
              xpath="//instance_attributes"/>
   
          <acl_permission id="minimal-deny-meta-attributes" kind="deny"
              xpath="//meta_attributes"/>
   
          <acl_permission id="minimal-deny-operations" kind="deny"
              xpath="//operations"/>
   
          <acl_permission id="minimal-deny-utilization" kind="deny"
              xpath="//utilization"/>
   
          <acl_permission id="minimal-nodes" kind="read"
              description="allow reading node names/IDs (attributes are denied separately)"
              xpath="/cib/configuration/nodes"/>
   
          <acl_permission id="minimal-resources" kind="read"
              description="allow reading resource names/agents (parameters are denied separately)"
              xpath="/cib/configuration/resources"/>
   
          <acl_permission id="minimal-deny-constraints" kind="deny"
              xpath="/cib/configuration/constraints"/>
   
          <acl_permission id="minimal-deny-topology" kind="deny"
              xpath="/cib/configuration/fencing-topology"/>
   
          <acl_permission id="minimal-deny-op_defaults" kind="deny"
              xpath="/cib/configuration/op_defaults"/>
   
          <acl_permission id="minimal-deny-rsc_defaults" kind="deny"
              xpath="/cib/configuration/rsc_defaults"/>
   
          <acl_permission id="minimal-deny-alerts" kind="deny"
              xpath="/cib/configuration/alerts"/>
   
          <acl_permission id="minimal-deny-acls" kind="deny"
              xpath="/cib/configuration/acls"/>
   
          <acl_permission id="minimal-cib" kind="read"
              description="allow reading cib element and crm_config/status sections"
              xpath="/cib"/>
   
      </acl_role>
   
      <acl_target id="alice">
         <role id="minimal"/>
      </acl_target>
   
      <acl_target id="bob">
         <role id="read_all"/>
      </acl_target>
   
      <acl_target id="carol">
         <role id="read_all"/>
         <role id="operator"/>
      </acl_target>
   
      <acl_target id="dave">
         <role id="administrator"/>
      </acl_target>
   
   </acls>
   ----
   
   In the above example, the user +alice+ has the minimal permissions necessary to
   run basic Pacemaker CLI tools, including using `crm_mon` to view the cluster
   status, without being able to modify anything. The user +bob+ can view the
   entire configuration and status of the cluster, but not make any changes. The
   user +carol+ can read everything, and change selected cluster properties as
   well as resource roles and location constraints. Finally, +dave+ has full read
   and write access to the entire CIB.
   
   Looking at the +minimal+ role in more depth, it is designed to allow read
   access to the +cib+ tag itself, while denying access to particular portions of
   its subtree (which is the entire CIB).
   
   This is because the DC node is indicated in the +cib+ tag, so `crm_mon` will
   not be able to report the DC otherwise. However, this does change the security
   model to allow by default, since any portions of the CIB not explicitly denied
   will be readable. The +cib+ read access could be removed and replaced with read
   access to just the +crm_config+ and +status+ sections, for a safer approach at
   the cost of not seeing the DC in status output.
   
   For a simpler configuration, the +minimal+ role allows read access to the
   entire +crm_config+ section, which contains cluster properties. It would be
   possible to allow read access to specific properties instead (such as
   +stonith-enabled+, +dc-uuid+, +have-quorum+, and +cluster-name+) to restrict
   access further while still allowing status output, but cluster properties are
   unlikely to be considered sensitive.
