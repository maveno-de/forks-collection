# Ansible Collection - maveno_de.forks

Documentation for the collection.

# Managing user

The role can be executed as root of the managed host or a specific user may be used, that is required to be able to perform passwordless sudo commands. A vendor label may be defined that will be used in prefixes for the used paths. These variables are allowed to be defined on group level, host level or be omitted.

```yaml
forksVendorIdentifier: <Vendor Identifier>
forksManagingSystemUsername: <Managing Systemuser Username>
forksManagingHomeDirectory: <Managing Systemuser Home Directory>
```

# Collection level variables

These variables need to be accessible by all roles of the collection. They should be placed in group_vars/all.yml or in the group vars section of the inventory file.

```yaml
forksManagingLocalBinaryDirectory: "{{ forksManagingHomeDirectory|default('/root') }}/.local/bin"

forksVariableDirectory: /var/local{% if forksVendorIdentifier is defined %}/{{ forksVendorIdentifier }}{% endif %}
forksUserRootDirectory: /var/local{% if forksVendorIdentifier is defined %}/{{ forksVendorIdentifier }}{% endif %}/lib

forksPrefixDirectory: "/usr/local{% if forksVendorIdentifier is defined %}/{{ forksVendorIdentifier }}{% endif %}"
forksSourceDirectory: "{{ forksPrefixDirectory }}/src"
forksLibraryDirectory: "{{ forksPrefixDirectory }}/lib"
forksLibrary64Directory: "{{ forksPrefixDirectory }}/lib64"
forksNodeBinaryDirectory: "{{ forksPrefixDirectory }}/bin"
forksLibexecDirectory: "{{ forksPrefixDirectory }}/libexec"
forksNodeSharedDirectory: "{{ forksPrefixDirectory }}/share"
forksSystemServiceDirectory: /etc/systemd/system

forksSystemPythonInterpreter: /usr/bin/python3
forksSystemCaCertificatesFilePath: /etc/ssl/certs/ca-certificates.crt

forksVenvDirectory: "/usr/local{% if forksVendorIdentifier is defined %}/{{ forksVendorIdentifier }}{% endif %}/share/{{ forksVendorIdentifier|default('forks') }}.venv"
forksVenvInterpreterPath: "{{ forksVenvDirectory }}/bin/python"
forksVenvPackages:
  - requests
  - packaging
  - docker


...
```
