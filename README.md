<h1>StorSimple Evacuation</h1>
<h2>Low imapact migration of StorSimple data to Azure Files</h2>

<p>
<h2>Introduction</h2>
StorSimple 8000 mainstream support will <a href="https://support.microsoft.com/en-us/lifecycle/search/19605">end on July 1 2020</a>.
A good alternative for StorSimple is the use of <a href="https://docs.microsoft.com/en-us/azure/storage/files/storage-files-introduction">Azure Files</a> in combination with <a href="https://www.youtube.com/watch?v=Zm2w8-TRn-o">Azure File Sync</a>. This gives a similar solution to StorSimple:
<br>
<li>Tiered file storage for on-premise use</li>
<li>Backup in Azure via snapshots of the file share</li>
<br>
Besides these cool features, a smart migration method from StorSimdple to Azure Files was developed by the Azure Files product team and <a href="https://myignite.techcommunity.microsoft.com/sessions/84177?source=sessions">presented at Ignite 2019</a>.
This migration method is "smart" because:
<br>
<li>It is has, besides a very short "switch over moment", no impact on the ongoing StorSimple 8000 operation.</li>
<li>It is done in Azure using virtual resources that can be discared after the migration.</li>
<br>
<br>
<h2>This document guides you quickly through this procedure.</h2>
<br>
<br>
<img src="https://github.com/joostm1/storsimple-exit/blob/master/content/storsimple-files-migration-overview.png" alt="Migration overview">
</p>


<p>
<h2>Steps in the migration process</h2>
<ol>
    <li>Create a Server 2019 file server with the iSCSI client</li>
    <li>In Azure, create a StorSimple Virtual Appliance and assign the snapshots to it</li>
    <li>Configure iSCSI and Azure File Sync on the file server</li>
    <li>Cutover day; sync the last changes and switch to Azure Files + Sync</li>
</ol>
These steps are outlined in the paragraphs below using the following naming scheme:
<pre><code>
    storsimple-on-premise
    storsimple-azure
    syncserver-azure
    syncserver-on-premise
</pre></code>
</p>

<p>
<br>
<h2>1. Create a Server 2019 server from the market place.</h2>
<br>
In Azure, create a file server using a Server 2019 image from the marketplace.
<ul>
    <li>Create this server in the same region as your StorSimple Device Manager is located.</li>
    <li>Anything above 4 cores and 32GB of RAM will do.</li>
    <li>Add an additional 512GB disk to the server that will serve later for hosting the sync groups.</li>
    <li>Configure the iSCSI client on this <code>syncserver-azure</code></li>
    <ul>
        <li>Go to "Server Manager", select "Tools" and select "iSCSI initiator"</li>
        <img src="https://github.com/joostm1/storsimple-exit/blob/master/content/iscsi-initiator.png"></li>
        <br>
        <li>Go the properties tab of the iSCSI initiator and copy the iSCSI Qualified Name (iqn) of your sync server.
        <img src="https://github.com/joostm1/storsimple-exit/blob/master/content/isci-iqn.png">
        You'll need to add this iqn to the iSCSI volumes we'll create later.</li>
    </ul>
</ul>    
</p>

<p>
<br>
<h2>2. Create a StorSimple Virtual Appliance and assign snapshots to it</h2>
<br>
<ul>
    <li>Create a number of StorSimple 8010 or 8020 devices as <a href="https://docs.microsoft.com/en-us/azure/storsimple/storsimple-8000-cloud-appliance-u2">per documentation</a></li>
  <ul>
    <li>Create this device with same device manager as the one that manages the physical StorSimple devices</li>
    <li>A single 8020 device can manage up to 64TB of capacity. Create as many 8020 devices as needed to manage all the capacity of your physical devices</li>
  </ul>
<li>Create clones of the volumes and assign them to one of the <code>storsimple-azure</code> devices just created.
<img src="https://github.com/joostm1/storsimple-exit/blob/master/content/clone-to-8020.png"></li>
    <ul>
        <li>Assign the iqn of our <code>syncserver-azure</code> to each of the volumes you create.
        <img src="https://github.com/joostm1/storsimple-exit/blob/master/content/assign-iqn.png"></li>
    </ul>
</ul>
</p>

<p>
<br>
<h2>3. Configure iSCSI and Azure File Sync on the file server</h2>
Now it is time to go to your new <code>syncserver-azure</code> and configure the newly assigned volumes.
<ul>
<li>Launch the "iSCSI Initiator" and add <code>storsimple-azure</code> as a target portal.
<img src="https://github.com/joostm1/storsimple-exit/blob/master/content/iscsi-target.png"></li>
<li>Auto configure all volumes that are assigned to the syncserver-azure iqn.
<img src="https://github.com/joostm1/storsimple-exit/blob/master/content/iscsi-autoconfigure.png"></li>
<li>Launch <b>diskmgmt.msc</b> and mount each volume in a folder corresponding to the volume name. For example:
<pre><code>
F:\sgs\vol0
F:\sgs\vol1
F:\sgs\vol2
</pre></code>
<img src="https://github.com/joostm1/storsimple-exit/blob/master/content/volume-sgmount.png">
</li>
</ul>

Deploy Azure File Sync on <code>syncserver-azure</code> as described <a href="https://docs.microsoft.com/en-us/azure/storage/files/storage-sync-files-deployment-guide?tabs=azure-portal">here.</a>

For <b>Cloud Endpoint</b>, you have a choice between <a href="https://azure.microsoft.com/en-us/pricing/details/storage/files/">"Premium and Standard"</a> storage. For larger deployments, premuum storage is recommended.</li>.

Specify the mounted volumes as <b>Server Endpoint</b> for File Sync:
<pre><code>
F:\sgs\vol0
F:\sgs\vol1
F:\sgs\vol2
</pre></code>

The sync agent will transfer all data from the mounted volumes to Azure Files. 

The environment we've build by now should look a bit like this one.
<img src="https://github.com/joostm1/storsimple-exit/blob/master/content/migration-overview.png">
</p>


<p>
<h2>4. Cutover day; sync the last changes and switch to Azure Files + Sync</h2>
Depending on the size / # files, the initial data transfer to Azure Files may take multiple weeks.
Wait for sync to completely move all files from the vm to the Azure file share. You can verify that sync is complete with the following steps:
<ol>
<li>Open the Event Viewer and navigate to Applications and Services</li>
<li>Navigate and open Microsoft\FileSync\Agent\Telemetry</li>
<li>Look for the most recent event 9102, which corresponds to a completed sync session</li>
<li>Select Details and confirm that the SyncDirection value is Upload</li>
<li>Check the both the HResult and the PerItemErrorCount and confirm that they are both 0.</li>
</ol>
This means that the sync session was successful. If HResult is a non-zero value, then there
was an error during sync. If the PerItemErrorCount is greater than 0 then some files or
folders did not sync properly. It is possible to have an HResult of 0 but a
PerItemErrorCount that is greater than 0 so you must check both.

After initial sync completion, create clones of the latest StorSimple backups and mount them again on these server endpoints.
<pre><code>
F:\sgs\vol0
F:\sgs\vol1
F:\sgs\vol2
</pre></code>
When this is also complete, decommission the StorSimple infrastructure and enjoy a much simpler life :-)
<img src="https://github.com/joostm1/storsimple-exit/blob/master/content/migration-overview-after.png">


