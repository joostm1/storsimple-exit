<h1>StorSimple Exit</h1>
<h2>Impactless migration of StorSimple data set to Azure Files</h2>

<h2>High level overview of the migration process</h2>
<ol>
    <li>Create a cloudsnapshot of the volumes that need migration</li>
    <li>In Azure, create a StorSimple Virtual Appliance and assign the snapshots to it</li>
    <li>Share these volumes via iSCSI to a Windows 2012 or -2016 file server</li>
    <li>Mount the iSCSI volunes on this file server</li>
    <li>Install the Sync Agent on this file server and use the volunes as sync target</li>
    <li>Sync Agent synchronizes all data between iSCSI volumes and Azure Files</li>
    <li>On cutover day; refresh the snapshots and Sync Agent efficientlt merges the latest deltas with the file share on Azure Files</li>
    <li>Files on Azure Files share can be shared to doamin joined users via a Windows File Server with the File Sync Agent on top of it. </li>
</ol>

