# Steps to reproduce CLI Issue 753 on the forcedotcom/cli project

1. execute the following command

    `sfdx force:org:create --setdefaultusername --wait 30 --setalias scratch-A --durationdays 1 --definitionfile config/a-project-scratch-def.json`

1. examine the contents of the temporary folder used to define the "shape" of the scratch org.

    It's contents are:
    ```
        /shape
            /objects
                Account.object
            package.xml 
    ```
    The `Account.object` file contains:
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <Object xmlns="http://soap.sforce.com/2006/04/metadata">
        <recordTypes>
            <fullName>Default</fullName>
            <label>Default</label>
            <active>true</active>
        </recordTypes>
    </Object>
    ```
    The `package.xml` file contains:
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <Package xmlns="http://soap.sforce.com/2006/04/metadata">
        <types>
            
            <members>Account</members>
            <name>CustomObject</name>
        </types>
        <types>
            
        <members>Account.Default</members>
            <name>RecordType</name>
        </types>

        <version>50.0</version>
    </Package>
    ```

1. execute the following command

    `sfdx force:org:create --setdefaultusername --wait 30 --setalias scratch-B --durationdays 1 --definitionfile config/b-project-scratch-def.json`

1. examine the contents of the temporary folder used to define the "shape" of the scratch org.

    It's contents are:
    ```
        /shape
            /objects
                Account.object
                Case.object
                Contact.object
            /settings
                FieldService.settings
            package.xml 
    ```
    The `Account.object` file remains the same as above.
    
    The `package.xml` file now contains:
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <Package xmlns="http://soap.sforce.com/2006/04/metadata">
        <types>
            
            <members>FieldService</members>
            <name>Settings</name>
        </types>
        <types>
            
            <members>Account</members>
            <members>Case</members>
            <members>Contact</members>
            <name>CustomObject</name>
        </types>
        <types>
            
        <members>Account.Default</members>
        <members>Case.Default</members>
        <members>Contact.Default</members>
            <name>RecordType</name>
        </types>
        <types>
            
        <members>Case.DefaultProcess</members>
            <name>BusinessProcess</name>
        </types>

        <version>50.0</version>
    </Package>
    ```
1. execute the first `force:org:create` command again.  This time with a different alias.

    `sfdx force:org:create --setdefaultusername --wait 30 --setalias scratch-FAILS --durationdays 1 --definitionfile config/a-project-scratch-def.json`

    The following failure message seen at this point:
    ```
    === Component Failures [4]
    TYPE   FILE                                  NAME                 PROBLEM
    ─────  ────────────────────────────────────  ───────────────────  ──────────────────────────────────────────────
    Error  shape/objects/Case.object             Case.Default         Not in package.xml
    Error  shape/objects/Case.object             Case.DefaultProcess  Not in package.xml
    Error  shape/objects/Contact.object          Contact.Default      Not in package.xml
    Error  shape/settings/FieldService.settings  FieldService         Not available for deploy for this organization
    ```

This error results from the fact that the "shape definition temp files" from the previous scratch org creation were not cleaned out at the beginning of the third execution of the `force:org:create` command.