## Here we'll basically understand how to install WinRM on a Windows Server Remotely

* So, here's an extension we need to configure while preparing a Terraform template

```tcl
################################################################################
# The Custom Script extension runs the publicly available Ansible script,
# that will configure a Windows Server 2019 Datacenter correctly to accept  
# secure WinRM over port 5986. This is required for Ansible.
################################################################################

resource "azurerm_virtual_machine_extension" "secure_winrm" {
  name                        = format("%s%s", local.display_name, "_cse_winrm")
  virtual_machine_id          = azurerm_windows_virtual_machine.cmp.id
  publisher                   = "Microsoft.Compute"
  type                        = "CustomScriptExtension"
  type_handler_version        = "1.10"
  auto_upgrade_minor_version  = true

  protected_settings = <<PROTECTED_SETTINGS
    {
      "commandToExecute": "powershell -ExecutionPolicy Unrestricted -File ConfigureRemotingForAnsible.ps1",
      "fileUris": [
        "${var.ansible_winrm_file_uri}"
      ]
    }
  PROTECTED_SETTINGS 

  depends_on = [
    azurerm_windows_virtual_machine.cmp,
    azurerm_virtual_machine_data_disk_attachment.cmp_disk_attach1,
    azurerm_site_recovery_replicated_vm.cmp_replication
  ]

  lifecycle {
    ignore_changes = all
  }
}
```

* Here, we're using the Custom Script extension to run a PowerShell script that will configure the Windows Servers

* Also, one thing to note is we're using a variable called _`var.ansible_winrm_file_uri`_ so this variable will be able to store the *`ConfigureRemotingForAnsible.ps1`* value 


## References ~

> _**[ConfigureRemotingForAnsible.ps1](https://raw.githubusercontent.com/ansible/ansible-documentation/c84880386a2f123ad5ee999bccfea4a502868663/examples/scripts/ConfigureRemotingForAnsible.ps1){:target="_blank"}**_