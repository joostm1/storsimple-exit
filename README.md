<h1>StorSimple Exit</h1>
<h2>Low imapact migration of StorSimple data to Azure Files</h2>

<h2>Introduction</h2>
StorSimple 8000 mainstream support will <a href="https://support.microsoft.com/en-us/lifecycle/search/19605">end on July 1 2020</a>.
A good alternative for StorSimple is the use of <a href="https://docs.microsoft.com/en-us/azure/storage/files/storage-files-introduction">Azure Files</a> in combination with <a href="https://www.youtube.com/watch?v=Zm2w8-TRn-o">Azure File Sync</a>. This gives a similar solution to StorSimple:
<li>Tiered file storage</li>
<li>Backup in Azure via snapshots of the file share</li>

Besided these cool features, a smart migration method from StorSimdple to Azure Files was developed by the Azure Files product team and <a href="https://myignite.techcommunity.microsoft.com/sessions/84177?source=sessions">presented at Ignite 2020</a>.
This migration method is "smart" because:
<li>It is has, besides a very short "switch over moment", no impact on the StorSimple 8000 operation.</li>
<li>It is done in Azure using virtual resources that can be discared after the migration.</li>

<b>This migration method is guided in detail in this document to make this migration as easy as possible for you</b> 

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

