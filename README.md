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
Besides these cool features, a smart migration method from StorSimdple to Azure Files was developed by the Azure Files product team and <a href="https://myignite.techcommunity.microsoft.com/sessions/84177?source=sessions">presented at Ignite 2020</a>.
This migration method is "smart" because:
<br>
<li>It is has, besides a very short "switch over moment", no impact on the ongoing StorSimple 8000 operation.</li>
<li>It is done in Azure using virtual resources that can be discared after the migration.</li>
<br>
<br>
<h2>The goal of this document is to guide you through this migration with little as possible effort.</h2>
<br>
<br>
<img src="https://github.com/joostm1/storsimple-exit/blob/master/content/storsimple-files-migration-overview.png" alt="Migration overview">
</p>


<h2>Steps in the migration process</h2>
<ol>
    <li>Create a Server 2019 file server with the iSCSI client</li>
    <li>In Azure, create a StorSimple Virtual Appliance and assign the snapshots to it</li>
    <li>Install the Sync Agent on this file server and use the volumes as sync target</li>
    <li>Cutover day; sync the last changes and switch to Azure Files + Sync</li>
</ol>
These steps are outlined in the paragraphs below using the following naming scheme:
<pre><code>
    storsimple-on-premise
    storsimple-azure
    syncserver-azure
    syncserver-on-premise
</pre></code>

<h2>Create a Server 2019 server from the market place.</h2>
In Azure, create a file server using a Server 2019 image from the marketplace.
<li>Create this server in the same region as your StorSimple Device Manager is located.<li>
<li>Anything above 4 cores and 32GB of RAM will do.<li>
<li>Add an additional 512GB disk to the server that will serve later for hosting the syncgroups.
<li>Configure the iSCSI client on this <code>syncserver-azure</code>
<lu>
    <li>Go to "Server Manager", select "Tools" and select "iSCSI initiator"br
    <img src=<li>
    
<p>
<h2>


<p>
<h2>Create a StorSimple Virtual Appliance and assign snapshots to it</h2>
<br>
<li>Create a StorSimple 8010 or 8020 device as <a href="https://docs.microsoft.com/en-us/azure/storsimple/storsimple-8000-cloud-appliance-u2">per documentation</a></li>
  <ul>
    <li>Create this device with device manager that also manages the physical StorSimple devices</li>
    <li>A single 8020 device can manage 64TB of capacity. Create as many 8020 devices as needed to manage all the capacity of your physical devices</li>
</ul>
<li>Create clones of the volumes and assign them to <code>storsimple-azure</clode> the just created.<li>
<li><img src="https://github.com/joostm1/storsimple-exit/blob/master/content/clone-to-8020.png"><li>



</p>

<p>
<h2>Share these volumes via iSCSI to a Windows 2012 or -2016 file server</h2>
<h2>Install the Sync Agent on this file server and use the volumes as sync target</h2>
<h2>Cutover day; sync the last changes and switch to Azure Files + Sync</h2>
