<p>This guide focuses on software configuration recommendations for users who are already familiar with the Intel&reg; Select Solutions for Genomics Analytics. The following guide contains the hardware configuration for the HPC clusters that are used to run this software:&nbsp;<em>HPC Cluster Tuning on 3rd Generation Intel&reg; Xeon&reg; Scalable Processors</em>. However, please carefully consider all of these settings based on your specific scenarios. Intel&reg; Select Solutions for Genomics Analytics can be deployed in multiple ways and this is a reference to one use-case.</p>

<p>Genomics analysis is accomplished using a suite of software tools including the following:</p>

<ul>
<li>Genomic Analysis Toolkit (GATK) Version 4.1.9.0 is used for variant discovery. GATK is the industry standard for identifying SNPs and indels in germline DNA and RNAseq data. It includes utilities to perform tasks such as processing and quality control of high-throughput sequencing data.</li>
<li>Cromwell Version 52 is a Workflow Management System for scientific workflows. These workflows are described in the Workflow Description Language (WDL). Cromwell tracks workflows and stores the output of tasks in a MariaDB database.</li>
<li>Burrows-Wheeler Aligner (BWA) Version 0.7.17 is used for mapping low-divergent sequences against a large reference genome, such as the human genome.</li>
<li>Picard Version 2.23.8 is set of tools used to manipulate high-throughput sequencing (HTS) data stored in various file formats such as: SAM, BAM, CRAM, or VCF.</li>
<li>VerifyBAMID2 works with Samtools Version 1.9 to verify whether sample genomic data matches previously known genotypes. It can also detect sample contamination.</li>
<li>Samtools Version 1.9 are used to interact with high-throughput sequencing data including: reading, writing, editing, indexing, or viewing data stored in the SAM, BAM, or CRAM format. It is also used for reading or writing BCF2, VCF, or gVCF files and for calling, filtering, or summarizing SNP and short indel sequence variants.</li>
<li>20K Throughput Run is a simple and quick benchmark test used to ensure that the most important functions of your HPC cluster are configured correctly.</li>
<li>Optional: Intel&reg; System Configuration Utility (SYSCFG) Version 14.2 Build 8 command-line utility can be used to save and to restore system BIOS and management firmware settings.</li>
</ul>

<p>These prerequisites are required:</p>

<ul>
<li>Git is a version control system for tracking changes.</li>
<li>Either Java 8 or the Java Runtime Environment (JRE) plus the Software Developers Kit (SDK) 1.8 is required to run Cromwell.</li>
<li>Python Version 2.6 or greater is required for the gatk-launch. Newer tools and workflows require Python 3.6.2 along with other Python packages. Use the Conda package manager to manage your environment and dependencies.</li>
<li>Slurm Workload Manager provides a means of scheduling jobs and allocating cluster resources to those jobs. For a detailed description of Slurm, please visit:&nbsp;<a href="https://slurm.schedmd.com/documentation.html">https://slurm.schedmd.com/documentation.html</a>.</li>
<li>sbt is required to compile Cromwell.</li>
<li>MariaDB is used by Cromwell for persistent storage.</li>
<li>R, Rscript, gsalib, ggplot2, reshape, and gplots are used by GATK to produce plots.</li>
</ul>

<p><strong>3rd Generation Intel</strong><sup>&reg;</sup><strong>&nbsp;Xeon</strong><sup>&reg;</sup><strong>&nbsp;Scalable processors</strong>&nbsp;deliver platforms with built-in AI acceleration that can be optimized for many workloads. They provide a performance foundation to help speed data&rsquo;s transformative impact, from a multi-cloud environment to the intelligent edge and back. Improvements of particular interest to this workload are:</p>

<ul>
<li>Enhanced Performance</li>
<li>Increased DDR4 Memory Speed &amp; Capacity</li>
<li>Intel<sup>&reg;</sup>&nbsp;Advanced Vector Extensions</li>
<li>Intel&reg; Genomics Kernel Library with AVX-512</li>
</ul>

<h3><a id="_Toc92729777"></a><a id="_Toc65839472"></a>Architecture</h3>

<p>Scientists use tools such as, the Genomic Analysis Toolkit, along with WDL scripts to process DNA and RNA data. Cromwell is used to manage the workflow. SLURM is used to schedule jobs and to allocate cluster resources to those jobs. This diagram shows the software and hardware used in the Intel&reg; Select Solutions for Genomics Analytics.</p>

<p style="text-align:center"><img alt="Genomics Architecture" height="382" src="/content/dam/develop/external/us/en/images/Genomics-Architecture-diagram.jpg" width="700"/></p>

<p><a id="_Toc65839458"></a><a id="_Toc65839473"></a></p>

<h2><a id="_Toc92729778"></a>Tuning the Intel&reg; Select Solutions for Genomics Analytics</h2>

<p>Software configuration tuning is essential because the operating system and the genomics analysis software were designed for general-purpose applications. The default configurations have not been tuned for the best performance. The following sections provide step-by-step guidance for tuning the Intel&reg; Select Solutions for Genomics Analytics.</p>

<h3><a id="_Toc92729779"></a>Tuning the HPC Cluster</h3>

<p>Tuning recommendations for hardware are addressed in this guide:&nbsp;<em>HPC Cluster Tuning on 3rd Generation Intel&reg; Xeon&reg; Scalable Processors</em>&nbsp;at: https://www.intel.com/content/www/us/en/developer/articles/guide/hpc-cluster-tuning-on-3rd-generation-xeon.html</p>

<h3><a id="_Toc92729780"></a>Configuring the Slurm Workload Manager</h3>

<p>The Slurm Workload Manager provides a means of scheduling jobs and allocating cluster resources to those jobs. When adding the Slurm workload manager to the HPC Cluster, install the server component on the frontend node.</p>

<h3><a id="3.9.1_Install_the_Slurm_Server_on_the_fr"></a><a id="_bookmark80"></a><a id="_Toc92729781"></a><a id="_Hlk87876856"></a>Installing the Slurm Server on the frontend node</h3>

<p>Via Slurm, the PAM (pluggable authentication module) restricts normal user SSH access to compute nodes. This may not be ideal in certain circumstances. Slurm needs a system user for the resource management daemons. The global Slurm configuration file and the cryptographic key that is required by the munge authentication library must be available on every host in the resource management pool. The default configuration supplied with the OpenHPC build of Slurm requires the &quot;slurm&quot; user account. Create a user group and grant unrestricted SSH access. Add the slurm user to this user group as follows:</p>

<li>Create a user group that is granted unrestricted SSH access:
</li>

<pre><code class="language-bash">groupadd sshallowlist</code>
</pre>

<li>Add the dedicated Intel&reg; Cluster Checker user to the whitelist. This user account should be able to run the Cluster Checker both inside and outside of a resource manager job.</li>

<pre><code class="language-bash">usermod -aG sshallowlist clck</code></pre>

<li>Create the Slurm user account:</li>

<pre><code class="language-bash">useradd --system --shell=/sbin/nologin slurm</code></pre>

<li>Install Slurm server packages:</li>

<pre><code class="language-bash">dnf -y install ohpc-slurm-server</code></pre>

<li>Update the Warewulf files:</li>

<pre><code class="language-bash">wwsh file sync</code></pre>

<h4>Summary of the commands</h4>

<p>Here are all the commands for this section:</p>

<pre>
<code class="language-bash">groupadd sshallowlist
usermod -aG sshallowlist clck
useradd --system --shell=/sbin/nologin slurm
dnf -y install ohpc-slurm-server
wwsh file sync</code></pre>

<h3><a id="3.9.2__Update_node_resource_information."></a><a id="_bookmark81"></a><a id="_Toc92729782"></a><a id="_Hlk87878925"></a>Updating node resource information</h3>

<p>Update the Slurm configuration file with the node names of the compute nodes, the properties of their processors, and the Slurm partitions or queues that are associated with your HPC Cluster, including:</p>

<ul>
<li>The NodeName tag, or tags, must reflect the names of the compute nodes along with a definition of their respective capabilities.</li>
<li>The Sockets tag defines the number of sockets in a compute node.</li>
<li>The CoresPerSocket tag defines the number of CPU cores per socket.</li>
<li>The ThreadsPerCore tag defines the number of threads that can be executed in parallel on a core.</li>
<li>The PartitionName tag, or tags, defines the Slurm partitions, or queues, where compute names are assigned. Users use these tags to access these resources. The name of the partition will be visible to the user.</li>
<li>The Nodes argument defines the compute nodes that belong to a given partition.</li>
<li>The Default argument defines which partition is the default partition. The default partition is the partition that is used when a user doesn&#39;t explicitly specify a partition.</li>
</ul>

<p>Complete the following tasks to create a Slurm configuration file for a cluster based on the Bill of Materials for this Reference Design:</p>

<ul>
<li>Use the openHPC template to create a new Slurm configuration file.</li>
<li>Add the host name of the frontend host as the ControlMachine to the Slurm configuration file.</li>
<li>Ensure that the SLURM control daemon will make a node available again when the node registers with a valid configuration after being DOWN</li>
<li>Update the NodeName definition to reflect the hardware capabilities</li>
<li>Update the PartitionName definition</li>
</ul>

<h4>Step-by-step instructions</h4>

<ol>
<li>To create a new SLURM config file, copy the template for the openHPC SLURM config file:
</li>

<pre><code class="language-bash">cp /etc/slurm/slurm.conf.ohpc /etc/slurm/slurm.conf</code></pre>

<li>Open the&nbsp;<em>/etc/slurm/slurm.conf&nbsp;</em>file and make the following changes:
</li>

<li>Locate and update the line beginning with &quot;ControlMachine&quot; to:<br />
ControlMachine=frontend
</li>

<li>The SLURM configuration that ships with OpenHPC has a default set-up. The SLURM control daemon will only make a node available after it goes into the DOWN state if the node was in the DOWN state because it was non-responsive. You can refer to the documentation on the ReturnToService configuration option in:&nbsp;<a href="https://slurm.schedmd.com/slurm.conf.html">https://slurm.schedmd.com/slurm.conf.html</a><br />

<br />
This configuration is reasonable for a large cluster under constant supervision by a system administrator. For a smaller cluster or a cluster that is not under constant supervision by a system administrator, a different configuration is preferable. SLURM should make a compute node available again when a node with a valid configuration registers with the SLURM control daemon. To enable this type of return to service, locate this line:<br />
<br />
ReturnToService=1<br />
<br />
Replace it with:<br />
ReturnToService=2</li>

<li>The SLURM configuration that ships with OpenHPC permits sharing a compute node if the specified resource requirements allow it. Adjust this so that jobs can be scheduled based on CPU and memory requirements.<br />
<br />
Locate the line:<br />
SelectType=select/cons_tres<br />
<br />
Replace it with the following. Take care to drop the &ldquo;t&rdquo; from tres and use the setting: select/cons_res:<br />
SelectType=select/cons_res<br />
<br />
Locate the line:<br />
SelectTypeParameters=CR_Core<br />
<br />
Replace it with:<br />
SelectTypeParameters=CR_Core_Memory</li>
<li>Update the node information to reflect the cluster configuration. Locate the NodeName tag. For compute nodes based on 3rd Generation Intel&reg; Xeon&reg; Scalable Gold 6348 processors, replace the existing line with the following line:</li>
</ol>

<p>NodeName=<strong>c[01-04</strong>] Sockets=2 CoresPerSocket=28 ThreadsPerCore=2RealMemory=480000 state=UNKNOWN</p>

<p><br />
When configured for use with Cromwell, bwa,all and haplo will be used to provide improved performance. Locate the PartitionName tag and replace the existing line with the following lines.</p>

<p>PartitionName=xeon Nodes=c[01-04] Priority=10000 default=YES MaxTime=24:00:00 State=UP</p>

<p>PartitionName=bwa Nodes=c[01-04] Priority=4000 Default=NO MaxTime=24:00:00 State=UP</p>

<p>PartitionName=all Nodes=c[01-04] Priority=6000 Default=NO MaxTime=24:00:00 State=UP</p>

<p>PartitionName=haplo Nodes=c[01-04] Priority=5000 default=NO MaxTime=24:00:00 State=UP</p>

<ol>
<li>The openHPC slurm.conf example has a typo that prevents slurmctld and slurmd from starting. Fix that typo by locating the following line and using a &ldquo;#&rdquo; symbol to make it a comment:<br />
<br />
Caution: A second line starting with JobCompType exists. DO NOT change that line!<br />
<br />
Locate this line:<br />
JobCompType=jobcomp/none<br />
<br />
Add the &ldquo;#&rdquo; in front to comment it, so that it reads as follows:<br />
#JobCompType=jobcomp/none</li>
<li>Save and close the file.</li>
<li>Import the new configuration files into Warewulf:
<pre>
<code class="language-bash">wwsh -y file import /etc/slurm/slurm.conf</code></pre>


</li>
<li>Update the&nbsp;<em>/etc/warewulf/defaults/provision.conf&nbsp;</em>file:
<ol>
<li>Open the /etc/warewulf/defaults/provision.conf file and located the line starting with:<br />
files = dynamic_hosts, passwd, group ...<br />
<br />
<em>Note: Do not change any part of the existing line. Append the following to the end of the line. Remember to include a comma at the beginning of the added text.</em><br />
<br />
, slurm.conf, munge.key</li>
<li>Save and close the file.</li>
</ol>
</li>
<li>Enable MUNGE and Slurm controller services on the frontend node:
<pre>
<code>syssystemctl enable slurmctld.service</code></pre>


</li>
</ol>

<h4>Summary of the commands</h4>

<p>Here are all the commands for this section:</p>

<pre>
<code class="language-bash">cp /etc/slurm/slurm.conf.ohpc /etc/slurm/slurm.conf
sed -i "s/^\(NodeName.*\)/#\1/" /etc/slurm/slurm.conf
echo "NodeName=${compute_prefix}[${first_node}-${last_node}] Sockets=${num_sockets} \
CoresPerSocket=${num_cores} ThreadsPerCore=2 State=UNKNOWN" &gt;&gt; /etc/slurm/slurm.conf
sed -i "s/ControlMachine=.*/ControlMachine=${frontend_name}/" /etc/slurm/slurm.conf
sed -i "s/^\(PartitionName.*\)/#\1/" /etc/slurm/slurm.conf
sed -i "s/^\(ReturnToService.*\)/#\1\nReturnToService=2/" /etc/slurm/slurm.conf
sed -i "s/^\(SelectTypeParameters=.*\)/#\1/" /etc/slurm/slurm.conf
sed -i "s/^\(JobCompType=jobcomp\/none\)/#\1/" /etc/slurm/slurm.conf

cat &gt;&gt; /etc/slurm/slurm.conf &lt;&lt; EOFslurm

PartitionName=xeon Nodes=${compute_prefix}[${first_node}-${last_node}] Default=YES MaxTime=24:00:00 State=UP
PartitionName=cluster Nodes=${compute_prefix}[${first_node}-${last_node}] Default=NO MaxTime=24:00:00 \
State=UP

EOFslurm

wwsh -y file import /etc/slurm/slurm.conf
sed -i "s/^files\(.*\)/files\1, slurm.conf, munge.key/" /etc/warewulf/defaults/provision.conf
syssystemctl enable slurmctld.service

</code></pre>

<h3><a id="3.9.3_Install_the_Slurm_Client_into_the_"></a><a id="_bookmark82"></a><a id="_Toc92729783"></a>Installing the Slurm Client into the compute node image</h3>

<ul>
<li>Add Slurm client support to the compute node image:</li>
</ul>

<pre>
<code class="language-bash">dnf -y --installroot=$CHROOT install ohpc-slurm-client</code></pre>

<ul>
<li>Create a munge.key file in the compute node image. It will be replaced with the latest copy at node boot time.&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;</li>
</ul>

<pre>
<code class="language-bash"> \cp -fa /etc/munge/munge.key $CHROOT/etc/munge/munge.key</code></pre>

<ul>
<li>Update the initial Slurm configuration on the compute nodes. The Warewulf synchronization mechanism does not synchronize during early boot.</li>
</ul>

<pre>
<code class="language-bash">\cp -fa /etc/slurm/slurm.conf $CHROOT/etc/slurm/</code></pre>

<ul>
<li>Enable the Munge and Slurm client services:</li>
</ul>

<pre>
<code class="language-bash">systemctl --root=$CHROOT enable munge.service</code></pre>

<ul>
<li>Allow unrestricted SSH access for the root user and the users in the sshallowlist group. Other users will be subject to SSH restrictions.</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;a. Open $CHROOT/etc/security/access.conf and append the following lines, in order, to the end of the file.</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; + : root : ALL</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; + : sshallowlist : ALL</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- : ALL : ALL</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;b. Save and close the file.</p>

<ul>
<li>Enable SSH control via the Slurm workload manager. Enabling PAM in the chroot environment limits SSH access to only those nodes where the user has active jobs.</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; a. Open $CHROOT/etc/pam.d/sshd and append the following lines, in order, to the end of the file:</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; account sufficient pam_access.so</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;account required pam_slurm.so</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;b. Save and close the file.</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;echo &quot;account sufficient pam_access.so&quot; &gt;&gt; $CHROOT/etc/pam.d/sshd</p>

<ul>
<li>Update the VNFS image:<br />
<br />
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;wwvnfs --chroot $CHROOT &ndash;hybridize<a id="3.9.4_Optional_-_Install_Slurm_Client_on"></a><a id="_bookmark83"></a></li>
</ul>

<h4>Summary of the commands</h4>

<p>Here are all the commands for this section:</p>

<pre>
<code class="language-bash">echo "+ : root : ALL"&gt;&gt;$CHROOT/etc/security/access.conf
echo "+ : sshallowlist : ALL"&gt;&gt;$CHROOT/etc/security/access.conf echo "- : ALL : ALL"&gt;&gt;$CHROOT/etc/security/access.conf
echo "account sufficient pam_access.so" &gt;&gt; $CHROOT/etc/pam.d/sshd
wwvnfs --chroot $CHROOT –hybridize</code></pre>

<h3><a id="_Toc92729784"></a><a id="_Hlk87878114"></a>Optional &ndash; Installing the Slurm client on the frontend node</h3>

<p>If you also want to use the cluster frontend node for compute tasks that are scheduled via Slurm, enable it as follows:</p>

<ul>
<li>Add Slurm client package to the frontend node:<br />
&nbsp; &nbsp; &nbsp; &nbsp; dnf -y install ohpc-slurm-client</li>
<li>Add the frontend node as a compute node to the SLURM:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; a. Open /etc/slurm/slurm.conf and locate the NodeName tag. For compute nodes based on 3rd Generation Intel&reg; Xeon&reg; Scalable Gold 6348 processors, replace the existing line with the following line. This should be one single line:</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;NodeName=frontend,<strong>c[01-04]&nbsp;</strong>Sockets=2 CoresPerSocket=28 ThreadsPerCore=2 State=UNKNOWN</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Locate the PartitionName tag and replace the existing line with these two lines as two single lines:</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; PartitionName=xeon Nodes=frontend,c[01-04] Default=YES MaxTime=24:00:00 State=UP PartitionName=cluster Nodes=frontend,c[01-04] Default=NO MaxTime=24:00:00 State=UP</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; b. Save and close the file.</p>

<ul>
<li>Restart the Slurm control daemon:<br />
&nbsp; &nbsp; &nbsp;systemctl restart slurmctld</li>
</ul>

<p>&nbsp;</p>

<ul>
<li>Enable and start the Slurm daemon on the frontend node:<br />
&nbsp; &nbsp; &nbsp; systemctl enable --now slurmd</li>
</ul>

<p>&nbsp;</p>

<ul>
<li>Update the initial Slurm configuration on the compute nodes:<br />
&nbsp; &nbsp; &nbsp; &nbsp;\cp -fa /etc/slurm/slurm.conf $CHROOT/etc/slurm/</li>
</ul>

<p>&nbsp;</p>

<ul>
<li>Update the VNFS image:<br />
&nbsp; &nbsp; &nbsp; wwvnfs --chroot $CHROOT --hybridize</li>
</ul>

<h4>Summary of the commands</h4>

<p>Here are all the commands in this section:</p>

<pre>
<code class="language-bash">sed -i "s/NodeName=/NodeName=frontend,/" /etc/slurm/slurm.conf
systemctl restart slurmctld
systemctl enable --now slurmd
\cp -fa /etc/slurm/slurm.conf $CHROOT/etc/slurm/
wwvnfs --chroot $CHROOT --hybridize</code></pre>

<h3><a id="3.9.5_Finalize_Slurm_configuration"></a><a id="_bookmark84"></a><a id="_Toc92729785"></a>Finalizing the Slurm configuration</h3>

<p>The resource allocation in Slurm version 20.11.X has been changed. This has direct implications for certain types of MPI based jobs. For details, see the Slurm release notes at:&nbsp;<a href="https://slurm.schedmd.com/archive/slurm-20.11.6/news.html">https://slurm.schedmd.com/archive/slurm-20.11.6/news.html</a></p>

<p>In order to allow applications to use the resource allocation method from Slurm version 20.02.X and earlier, set the &quot;SLURM_OVERLAP&quot; environment variable on the frontend node to allow all users:<br />
cat &gt;&gt; /etc/environment &lt;&lt; EOFSLURM export SLURM_OVERLAP=1 EOFSLURM</p>

<ul>
<li>Re-start the Munge and Slurm controller services on the frontend node:<br />
&nbsp; &nbsp; systemctl restart munge</li>
<li>Reboot the compute nodes:<br />
&nbsp; &nbsp; pdsh -w c[01-XX] reboot</li>
<li>Ensure that the compute nodes are available in Slurm after they have been re- provisioned. In order to ensure that the compute nodes are available for resource allocation in Slurm, they need to be put into the &quot;idle&quot; state.</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;scontrol reconfig</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;scontrol update NodeName=c[01-XX] State=Idle</p>

<ul>
<li>Check the Slurm status. All nodes should have the state &quot;idle&quot;.</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; [root@frontend ~]# sinfo</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; PARTITION AVAIL TIMELIMIT NODES STATE NODELIST</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; xeon* up 1-00:00:00 4 idle c[01-04]</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; cluster up 1-00:00:00 4 idle c[01-04]</p>

<h4>Summary of the commands</h4>

<p>Here are all the commands in this section:</p>

<pre>
<code class="language-bash">cat &gt;&gt; /etc/environment &lt;&lt; EOFSLURM export SLURM_OVERLAP=1 EOFSLURM
systemctl restart munge
pdsh -w c[01-XX] reboot
scontrol reconfig
scontrol update NodeName=c[01-XX] State=Idle
[root@frontend ~]# sinfo</code></pre>

<h3><a id="3.9.6_Run_a_Test_Job"></a><a id="_bookmark85"></a><a id="_Toc92729786"></a>Running a test</h3>

<p>Once the resource manager is in production, normal users should be able to run jobs. Test this by creating a &quot;test&quot; user with standard privileges. For example, this user will not be allowed to use SSH to interact with the compute nodes outside of a Slurm job. Compile and execute a &quot;hello world&quot; application interactively through the resource manager. Note the use of srun for parallel job execution because it summarizes the underlying, native job launch mechanism.</p>

<ul>
<li>Add a &quot;test&quot; user:<br />
&nbsp; &nbsp; useradd -m test</li>
<li>Create a password for the user:<br />
&nbsp; &nbsp; passwd test</li>
<li>Synchronize the files with the Warewulf database. <em>Note: The synchronization to the compute nodes can take several minutes.</em></li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;wwsh file resync</p>

<ul>
<li>Switch to the &quot;test&quot; user account:<br />
&nbsp; &nbsp;&nbsp;su &ndash; test</li>
<li>Source the environment for the MPI implementation that will be used. In this example, the Intel MPI provided by oneAPI was used.<br />
&nbsp; &nbsp;&nbsp;module load oneapi</li>
<li>Compile the MPI &quot;hello world&quot; example:<br />
&nbsp; &nbsp;&nbsp;mpicc -o test /opt/intel/oneapi/mpi/latest/test/test.c</li>
<li>Submit the job. Start a Slurm job on all nodes. This should produce output similar to the following:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; [test@frontend ~]# srun --mpi=pmi2 --partition=xeon -N 4 -n 4 /home/test/test</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Hello world: rank 0 of 4 running on c01.cluster</p>

<ul>
<li>When the MPI job rans successfully, exit as the &quot;test&quot; user and continue with the cluster setup:<br />
&nbsp; &nbsp; exit</li>
</ul>

<h2><a id="3.10_Install_the_Genomics_Software_Stack"></a><a id="_bookmark86"></a><a id="_Toc92729787"></a>Installing the genomics software stack</h2>

<p>Install the genomics software stack. This software stack is based on the&nbsp;<a href="https://gatk.broadinstitute.org/hc/en-us">Genome Analysis ToolKit (GATK)&nbsp;</a>suite of tools along with the&nbsp;<a href="https://cromwell.readthedocs.io/en/stable/">Cromwell workflow management system</a>.</p>

<h3><a id="3.10.1_Check_the_Setup_of_the_Genomics_C"></a><a id="_bookmark87"></a><a id="_Toc92729788"></a>Checking the setup of the genomics cluster environment</h3>

<p>First, check that the cluster setup used to run the Genomics solution is properly configured.</p>

<ul>
<li>Make sure that Cromwell users on every node are limited to opening no more than 500,000 files:<br />
&nbsp; &nbsp; pdsh -w frontend,c0[1-4] &quot;su - cromwell -c &#39;ulimit -n&#39;&quot;</li>
</ul>

<p><em>Note: This should be at least 500,000 on every node because workflow settings require many files to be open during runtime.</em></p>

<ul>
<li>Check that the&nbsp;<em>/genomics_local&nbsp;</em>is a mounted filesystem with the correct ownership and permissions. The pdsh command should produce output identical to the one shown, except the order of compute nodes may vary.</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;[root@frontend ~]# pdsh -w c0[1-4] &quot;stat -c &#39;%A %U:%G %m&#39; /genomics_local&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;c01: drwxr-xr-x cromwell:cromwell /genomics_local</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;c03: drwxr-xr-x cromwell:cromwell /genomics_local</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;c04: drwxr-xr-x cromwell:cromwell /genomics_local</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;c02: drwxr-xr-x cromwell:cromwell /genomics_local</p>

<h3><a id="3.10.2_Configure_MariaDB"></a><a id="_bookmark88"></a><a id="_Toc92729789"></a>Configuring MariaDB</h3>

<p>Cromwell uses MariaDB for storing job information. Since Warewulf also uses MariaDB, the database may have been setup according to the HPC Cluster guide during the hardware configuration. If not, follow these instructions to set it up. You will need the MariaDB database administrator password.</p>

<ul>
<li>Enable the database service to start automatically at system boot<br />
&nbsp; &nbsp; systemctl enable mariadb.service</li>
<li>Enable the web server service to start automatically at system boot<br />
&nbsp; &nbsp; systemctl enable httpd.service</li>
<li>Restart the database service<br />
&nbsp; &nbsp; systemctl restart mariadb</li>
<li>Restart web services<br />
&nbsp; &nbsp; systemctl restart httpd</li>
<li>Update the Warewulf database password. This step must be done manually.Edit this file: /etc/warewulf/database-root.conf
<ul>
<li>We recommend that this password be different than the password for the root superuser.</li>
<li>Replace the &quot;changeme&quot; password with a password that you choose based on your password policy:<br />
&nbsp; &nbsp; &nbsp; &nbsp;database password = &lt;new_password&gt;</li>
</ul>
</li>
<li>Save and exit the file.</li>
<li>Secure database. MariaDB ships with a tool that allows the administrator to manually setup database access. This includes setting a password for the root user of the database. We recommend that this password be different from the password of the root user of the system and password for the Warewulf database administrator. Please follow your password policy when choosing passwords. Execute the following command and respond to the prompts. We strongly recommend using the default answers for all questions.<br />
&nbsp; &nbsp; &nbsp; /usr/bin/mysql_secure_installation</li>
<li>Initialize the Warewulf data store system. Enter the database root password when prompted.<br />
&nbsp; &nbsp; wwinit DATASTORE</li>
<li>Log into the MariaDB server as the database administrator and enter the root password when prompted:<br />
&nbsp; &nbsp; mysql -h localhost -u root -p</li>
</ul>

<p><em>Note: If you are unable to log into MariaDB see the&nbsp;<a href="https://word2aem.app.intel.com/#_bookmark98">Troubleshooting of the genomics software stack section of this guide&nbsp;</a>for suggestions</em></p>

<ul>
<li>Create a new database named &ldquo;cromwell&rdquo;:<br />
&nbsp; &nbsp; MariaDB [(none)]&gt; CREATE DATABASE cromwell;</li>
<li>Create a new MySQL database user. In this example, &ldquo;cromwell&rsquo; is also the password. Change the password to meet your password security policy.<br />
&nbsp; &nbsp; MariaDB [(none)]&gt; CREATE USER &#39;cromwell&#39;@&#39;localhost&#39; IDENTIFIED BY &#39;cromwell&#39;;</li>
<li>Grant database permissions to the new user. Replace &ldquo;cromwell&rdquo; with your password.<br />
&nbsp; &nbsp; MariaDB [(none)]&gt; GRANT ALL PRIVILEGES ON `cromwell`.* TO &#39;cromwell&#39;@&#39;localhost&#39;;</li>
<li>Reload and exit:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;MariaDB [(none)]&gt; FLUSH PRIVILEGES;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;MariaDB [(none)]&gt; exit</p>

<h3><a id="3.10.3_Installing_the_Cromwell_Workflow_"></a><a id="_bookmark89"></a><a id="_Toc92729790"></a><a id="_Hlk87879331"></a>Installing the Cromwell Workflow Management System</h3>

<p>Install the Cromwell Workflow Management system and configure it to use the local scratch device on the compute nodes.</p>

<ul>
<li>In order to install Cromwell, the &quot;sbt&quot; build tool is required. Install sbt:
<ul>
<li>Add the sbt online repository</li>
</ul>
</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;dnf config-manager --add-repo https://www.scala-sbt.org/sbt-rpm.repo</p>

<ul>
<li>Install sbt build tool:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;dnf -y install sbt</p>

<ul>
<li>Download Cromwell from the git repository:
<ul>
<li>Create a directory for installing Cromwell:</li>
</ul>
</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mkdir -p ${GENOMICS_PATH}/cromwell</p>

<ul>
<li>Set the owner of that directory to the&nbsp;<em>cromwell&nbsp;</em>user and the<em>&nbsp;cromwell</em>&nbsp;group. Change the ownership permissions in order to allow access.</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; chmod 775 -R ${GENOMICS_PATH}/Cromwell</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; chown -R cromwell:cromwell ${GENOMICS_PATH}/Cromwell</p>

<ul>
<li>Login to the cromwell user account:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;su - cromwell</p>

<ul>
<li>Use git to clone the Cromwell git repository:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; cd ${GENOMICS_PATH}/cromwell</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; git clone&nbsp;<a href="https://github.com/broadinstitute/cromwell.git">https://github.com/broadinstitute/cromwell.git</a></p>

<ul>
<li>This guide was tested and validated with version 52 of Cromwell. Checkout version 52 from the repository:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; cd cromwell</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;git checkout 52</p>

<ul>
<li>Configure Cromwell to use the local NVMe disk as scratch space:
<ul>
<li>Open this file for editing:</li>
</ul>
</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; backend/src/main/scala/cromwell/backend/RuntimeEnvironment.scala</p>

<ul>
<li>Comment out line 3, so it reads:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; //import java.util.UUID</p>

<ul>
<li>Update lines 23 through 27 as shown below:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;val tempPath: String = {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;val uuid = UUID.randomUUID().toString</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;val hash = uuid.substring(0, uuid.indexOf(&#39;-&#39;)) callRoot.resolve(s&quot;tmp.$hash&quot;).pathAsString</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }</p>

<ul>
<li>Add the following text in line 23:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; def tempPath: String = &quot;/genomics_local&quot;</p>

<ul>
<li>Save the file and exit</li>
<li>Open this file:
<ul>
<li>backend/src/main/scala/cromwell/backend/standard/ StandardAsyncExecutionActor.scala</li>
</ul>
</li>
<li>Go to line number 380 to find the following content:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|export _JAVA_OPTIONS=-Djava.io.tmpdir=&quot;$$tmpDir&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|export TMPDIR=&quot;$$tmpDir&quot;</p>

<ul>
<li>Replace those two lines with the following text:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|mkdir -p $$tmpDir/tmp.$$$$</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|export _JAVA_OPTIONS=-Djava.io.tmpdir=&quot;$$tmpDir/tmp.$$$$&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|export TMPDIR=&quot;$$tmpDir/tmp.$$$$</p>

<ul>
<li>Save the file and exit.</li>
<li>Build Cromwell with the patches:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; sbt clean</p>

<ul>
<li>When the build is successful, move the new jar file into the ${GENOMICS_PATH}/cromwell directory</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; cp server/target/scala-2.12/cromwell-52-*-SNAP.jar \</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ${GENOMICS_PATH}/cromwell/cromwell-52-fix.jar</p>

<h4>Summary of the commands</h4>

<p>Here are all the commands for this section</p>

<pre>
<code class="language-bash">sed -i "s/^\(import\ java.util.UUID\)/\/\/\1/" \
backend/src/main/scala/cromwell/backend/RuntimeEnvironment.scala
sed -i '23,27d' \ backend/src/main/scala/cromwell/backend/RuntimeEnvironment.scala
sed -i '23i \ \ \ \ \ \ def tempPath: String = \"/genomics_local\"' \ backend/src/main/scala/cromwell/backend/RuntimeEnvironment.scala
sed 's/\(\s*|export _JAVA_OPTIONS.*\)\"/\ \ \ \ \ \ \ \ |mkdir -p \$\$tmpDir\/tmp\.\$\$\$\$\n\1\"/' \ backend/src/main/scala/cromwell/backend/standard/StandardAsyncExecutionActor.scala
sed 's/\(\s*|export _JAVA_OPTIONS.*tmpDir\)\"/\1\/tmp\.\$\$\$\$\"/' \ backend/src/main/scala/cromwell/backend/standard/StandardAsyncExecutionActor.scala
sed 's/\(\s*|export TMPDIR=.*tmpDir\)\"/\1\/tmp\.\$\$\$\$\"/' \ backend/src/main/scala/cromwell/backend/standard/StandardAsyncExecutionActor.scala</code></pre>

<p>&nbsp;</p>

<ul>
<li>Edit backend/src/main/scala/cromwell/backend/standard/ StandardAsyncExecutionActor.scala</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Replace line 380-381 with:</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; |mkdir -p $$tmpDir/tmp.$$$$</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; |export _JAVA_OPTIONS=-Djava.io.tmpdir=&quot;$$tmpDir/tmp.$$$$&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; |export TMPDIR=&quot;$$tmpDir/tmp.$$$$</p>

<ul>
<li>Then continue executing:</li>
</ul>

<pre>
<code class="language-bash">sbt clean
cp server/target/scala-2.12/cromwell-52-*-SNAP.jar ${GENOMICS_PATH}/cromwell/cromwell-52-fix.jar</code></pre>

<h3><a id="3.10.4_Configure_the_Execution_Environme"></a><a id="_bookmark90"></a><a id="_Toc92729791"></a>Configuring the execution environment for Cromwell</h3>

<p>Configure the Cromwell execution environment. Use the default configuration file as a starting point and change it to reflect your needs .</p>

<ul>
<li>Configure MariaDB as the database for Cromwell to use</li>
<li>Add the SLURM backends, so Cromwell can schedule jobs using SLURM</li>
</ul>

<h4>Step-by-step instructions</h4>

<ul>
<li>First, download the default Cromwell&nbsp;<em>reference.conf&nbsp;</em>configuration file.<br />
&nbsp; &nbsp; &nbsp; &nbsp;wget https://raw.githubusercontent.com/broadinstitute/cromwell/52_hotfix/core/\</li>
<li>Add MariaDB as the database for Cromwell to use.
<ul>
<li>Open this file:&nbsp;<em>${GENOMICS_PATH}/cromwell/reference.conf</em></li>
<li>Add the following lines at the end of the file, then save the file and exit:</li>
</ul>
</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;database {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;profile = &quot;slick.jdbc.MySQLProfile$&quot; db {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;driver = &quot;com.mysql.cj.jdbc.Driver&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;url = &quot;jdbc:mysql://localhost/cromwell?rewriteBatchedStatements=true&amp;serverTimezone=UTC&quot; user = &quot;cromwell&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;password = &quot;cromwell&quot; connectionTimeout = 5000</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}</p>

<p>.</p>

<ul>
<li>Add SLURM as the backend for Cromwell:
<ul>
<li>Open this file:&nbsp;<em>${GENOMICS_PATH}/cromwell/reference.conf</em></li>
<li>Go to line 479 to find:</li>
</ul>
</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;default = &quot;Local&quot;</p>

<ul>
<li>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Change&nbsp;<em>&quot;Local&quot;&nbsp;</em>to&nbsp;<em>&quot;SLURM&quot;</em></li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; default = &quot;SLURM&quot;</p>

<ul>
<li>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Remove the following five lines (line numbers 480 through 484)</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;providers {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Local {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; actor-factory = &quot;cromwell.backend.impl.sfs.config.ConfigBackendLifecycleActorFactory&quot;<br />
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; config {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; include required(classpath(&quot;reference_local_provider_config.inc.conf&quot;))</p>

<ul>
<li>Now add the following text after line 479, the line that has&nbsp;<em>default = &quot;SLURM&quot;.</em></li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<em> Note: Ensure that the lines that show line-breaks in this document are, in fact, single lines in reference.conf</em></p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; providers { SLURM {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # Modifying temp directory to write to local disk temporary-directory = &quot;$(/genomics_local/)&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; actor-factory = &quot;cromwell.backend.impl.sfs.config.ConfigBackendLifecycleActorFactory&quot; config {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; root = &quot;cromwell-slurm-exec&quot; runtime-attributes = &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Int runtime_minutes = 600 Int cpu = 2</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Int memory_mb = 1024 String queue = &quot;all&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String? docker &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; submit = &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; sbatch -J ${job_name} -D ${cwd} -o ${out} -e ${err} -t ${runtime_minutes} -p ${queue} ${&quot;-c &quot; + cpu} --mem ${memory_mb} --wrap &quot;/bin/bash ${script}&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; kill = &quot;scancel ${job_id}&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; check-alive = &quot;squeue -j ${job_id}&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; job-id-regex = &quot;Submitted batch job (\\d+).*&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; SLURM-BWA {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; temporary-directory = &quot;$(/genomics_local/)&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; actor-factory = &quot;cromwell.backend.impl.sfs.config.ConfigBackendLifecycleActorFactory&quot; config {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; root = &quot;cromwell-slurm-exec&quot; runtime-attributes = &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Int runtime_minutes = 600 Int cpu = 2</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Int memory_mb = 1024 String queue = &quot;bwa&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String? docker &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; submit = &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; sbatch -J ${job_name} -D ${cwd} -o ${out} -e ${err} -t ${runtime_minutes} -p ${queue} ${&quot;-c &quot; + cpu} --mem ${memory_mb} --wrap &quot;/bin/bash ${script}&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; kill = &quot;scancel ${job_id}&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; check-alive = &quot;squeue -j ${job_id}&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; job-id-regex = &quot;Submitted batch job (\\d+).*&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; SLURM-HAPLO {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; temporary-directory = &quot;$(/genomics_local/)&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; actor-factory = &quot;cromwell.backend.impl.sfs.config.ConfigBackendLifecycleActorFactory&quot; config {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; root = &quot;cromwell-slurm-exec&quot; runtime-attributes = &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Int runtime_minutes = 600 Int cpu = 2</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Int memory_mb = 1024 String queue = &quot;haplo&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String? docker &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; submit = &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; sbatch -J ${job_name} -D ${cwd} -o ${out} -e ${err} -t ${runtime_minutes} -p ${queue} ${&quot;-c &quot; + cpu} --mem ${memory_mb} --wrap &quot;/bin/bash ${script}&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &quot;&quot;&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; kill = &quot;scancel ${job_id}&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; check-alive = &quot;squeue -j ${job_id}&quot;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; job-id-regex = &quot;Submitted batch job (\\d+).*&quot;</p>

<ul>
<li>Save the file and exit.</li>
</ul>

<h4>Summary of the commands</h4>

<p>Here are all the commands in this section:</p>

<pre>
<code class="language-bash">sed -i '$ a database {' reference.conf
sed -i '$ a \ \ profile = \"slick.jdbc.MySQLProfile$\"' reference.conf sed -i '$ a \ \ db {' reference.conf
sed -i '$ a \ \ \ \ driver = \"com.mysql.cj.jdbc.Driver\"' reference.conf
sed -i '$ a \ \ \ \ url = \"jdbc:mysql:\/\/localhost\/cromwell\?rewriteBatched\ Statements=true&amp;serverTimezone=UTC\"' reference.conf
sed -i '$ a \ \ \ \ user = \"cromwell\"' reference.conf
sed -i '$ a \ \ \ \ password = \"cromwell\"' reference.conf sed -i '$ a \ \ \ \ connectionTimeout = 5000' reference.conf sed -i '$ a \ \ }' reference.conf
sed -i '$ a }' reference.conf
sed -i '479,484d' reference.conf
sed -i "479i \ \ \ \ \ \ \ \ job-id-regex\ =\ \"Submitted\ batch\ job\ (\\\\\\\\d+).*\"" \ reference.conf
sed -i "479i \ \ \ \ \ \ \ \ check-alive\ =\ \"squeue\ -j\ \${job_id}\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ kill\ =\ \"scancel\ \${job_id}\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ \"\"\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ sbatch\ -J\ \${job_name}\ -D\ \${cwd}\ -o\ \${out}\ -e\ \${err}\ \
-t\ \${runtime_minutes}\ -p\ \${queue}\ \${\"-c\ \"\ +\ cpu}\ --mem\ \${memory_mb}\ \
--wrap\ \"\/bin\/bash\ \${script}\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ submit\ =\ \"\"\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ \"\"\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ String?\ docker" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ String\ queue\ =\ \"haplo\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ Int\ memory_mb\ =\ 1024" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ Int\ cpu\ =\ 2" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ Int\ runtime_minutes\ =\ 600" reference.conf sed -i "479i \ \ \ \ \ \ \ \ runtime-attributes\ =\ \"\"\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ root\ =\ \"cromwell-slurm-exec\"" reference.conf sed -i "479i \ \ \ \ \ \ config\ {" reference.conf
sed -i "479i \ \ \ \ \ \ actor-factory\ =\ \"cromwell.backend.impl.sfs.config.\ ConfigBackendLifecycleActorFactory\"" reference.conf
sed -i "479i \ \ \ \ \ \ temporary-directory\ =\ \"\$(\/genomics_local\/)\"" reference.conf sed -i "479i \ \ \ \ SLURM-HAPLO\ {" reference.conf
sed -i "479i \ \ \ \ }\ \ " reference.conf sed -i "479i \ \ \ \ \ \ }" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ job-id-regex\ =\ \"Submitted\ batch\ job\ (\\\\\\\\d+).*\"" \ reference.conf
sed -i "479i \ \ \ \ \ \ \ \ check-alive\ =\ \"squeue\ -j\ \${job_id}\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ kill\ =\ \"scancel\ \${job_id}\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ \"\"\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ sbatch\ -J\ \${job_name}\ -D\ \${cwd}\ -o\ \${out}\ -e\ \${err}\ \
-t\ \${runtime_minutes}\ -p\ \${queue}\ \${\"-c\ \"\ +\ cpu}\ --mem\ \${memory_mb}\ \
--wrap\ \"\/bin\/bash\ \${script}\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ submit\ =\ \"\"\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ \"\"\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ String?\ docker" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ String\ queue\ =\ \"bwa\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ Int\ memory_mb\ =\ 1024" reference.conf sed -i "479i \ \ \ \ \ \ \ \ Int\ cpu\ =\ 2" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ Int\ runtime_minutes\ =\ 600" reference.conf sed -i "479i \ \ \ \ \ \ \ \ runtime-attributes\ =\ \"\"\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ root\ =\ \"cromwell-slurm-exec\"" reference.conf sed -i "479i \ \ \ \ \ \ config\ {" reference.conf
sed -i "479i \ \ \ \ \ \ actor-factory\ =\ \"cromwell.backend.impl.sfs.config.\ ConfigBackendLifecycleActorFactory\"" reference.conf
sed -i "479i \ \ \ \ \ \ temporary-directory\ =\ \"\$(\/genomics_local\/)\"" reference.conf sed -i "479i \ \ \ \ SLURM-BWA\ {" reference.conf
sed -i "479i \ \ \ \ }" reference.conf
sed -i "479i \ \ \ \ \ \ }\ \ " reference.conf
sed -i "479i \ \ \ \ \ \ \ \ job-id-regex\ =\ \"Submitted\ batch\ job\ (\\\\\\\\d+).*\"" \ reference.conf
sed -i "479i \ \ \ \ \ \ \ \ check-alive\ =\ \"squeue\ -j\ \${job_id}\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ kill\ =\ \"scancel\ \${job_id}\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ \"\"\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ sbatch\ -J\ \${job_name}\ -D\ \${cwd}\ -o\ \${out}\ -e\ \${err}\ \
-t\ \${runtime_minutes}\ -p\ \${queue}\ \${\"-c\ \"\ +\ cpu}\ --mem\ \${memory_mb}\ \
--wrap\ \"\/bin\/bash\ \${script}\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ submit\ =\ \"\"\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ \"\"\"" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ String?\ docker" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ String\ queue\ =\ \"all\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ Int\ memory_mb\ =\ 1024" reference.conf sed -i "479i \ \ \ \ \ \ \ \ Int\ cpu\ =\ 2" reference.conf
sed -i "479i \ \ \ \ \ \ \ \ Int\ runtime_minutes\ =\ 600" reference.conf sed -i "479i \ \ \ \ \ \ \ \ runtime-attributes\ =\ \"\"\"" reference.conf sed -i "479i \ \ \ \ \ \ \ \ root\ =\ \"cromwell-slurm-exec\"" reference.conf sed -i "479i \ \ \ \ \ \ config\ {" reference.conf
sed -i "479i \ \ \ \ \ \ actor-factory\ =\ \"cromwell.backend.impl.sfs.config.\ ConfigBackendLifecycleActorFactory\"" reference.conf
sed -i "479i \ \ \ \ \ \ temporary-directory\ =\ \"\$(\/genomics_local\/)\"" reference.conf
sed -i "479i \ \ \ \ \ \ #\ Modifying\ temp\ directory\ to\ write\ to\ local\ disk" reference.conf sed -i "479i \ \ \ \ SLURM\ {" reference.conf
sed -i "479i \ \ providers\ {" reference.conf
sed -i "479i \ \ default\ =\ \"SLURM\"" reference.conf</code></pre>

<h3><a id="3.10.5_Check_the_Cromwell_configuration"></a><a id="_bookmark91"></a><a id="_Toc92729792"></a>Validating the Cromwell configuration</h3>

<p>Validate that the Cromwell installation is working properly by executing a simple smoke test.</p>

<ul>
<li>Change to the &quot;cromwell&quot; user<br />
&nbsp; &nbsp; su - cromwell</li>
<li>Change to the Cromwell directory<br />
&nbsp; &nbsp; cd ${GENOMICS_PATH}/cromwell</li>
<li>Start the Cromwell server as a detached process in the background. By default the Cromwell server will be accessible via port 8000. The&nbsp;<em>cromwell.log&nbsp;</em>file will contain the messages the Cromwell server generates during execution.<br />
&nbsp; &nbsp; nohup java -jar -Dconfig.file=reference.conf cromwell-52-fix.jar server 2&gt;&amp;1 &gt;&gt;cromwell.log &amp;</li>
<li>Confirm that the Cromwell server is running by checking the process list by using the &quot;ps&quot; command:</li>
</ul>

<p><em>Note: The output of the &quot;ps&quot; command will output the PID and other information that we are not concerned with. We are using &lt;&hellip;&gt; as a placeholder for that information.</em><br />
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; [cromwell@frontend cromwell]$ ps aux | grep cromwell | grep server</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;cromwell &lt;PID&gt; &lt;&hellip;&gt; java -jar -Dconfig.file=reference.conf cromwell-52-fix.jar server</p>

<p><em>Note: In order to stop the Cromwell server, use the &quot;kill -9 &lt;PID&gt;&quot; command where &lt;PID&gt; is the process ID running Cromwell.</em></p>

<ul>
<li>Now that the Cromwell server process is running, execute an example workflow. This workflow was designed using the Workflow Description Language (WDL). It will ensure that Cromwell is configured correctly with the job scheduler.
<ul>
<li>Navigate to your working directory:<br />
&nbsp; &nbsp; cd ${GENOMICS_PATH}/cromwell/</li>
<li>Open a new file named&nbsp;<em>HelloWorld.wdl&nbsp;</em>and add the following text:</li>
</ul>
</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;task hello {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String name command {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;echo &#39;Hello ${name}!&#39;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;output {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;File response = stdout()</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; runtime {</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;memory: &quot;2MB&quot; disk: &quot;2MB&quot;</p>

<p>&nbsp;</p>

<ul>
<li>Save the file and exit.</li>
<li>Open a new file called:&nbsp;<em>HelloWorld.json</em></li>
<li>Add the following text to that file:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {&quot;helloWorld.hello.name&quot;: &quot;World&quot;}</p>

<ul>
<li>Save the file and exit.</li>
<li>We can now submit the workflow to the Cromwell server using the WDL and JSON input files that were just created:<br />
<br />
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;curl -v http://127.0.0.1:8000/api/workflows/v1 -F \ workflowSource=@${GENOMICS_PATH}/cromwell/HelloWorld.wdl -F \ workflowInputs=@${GENOMICS_PATH}/cromwell/HelloWorld.json</li>
<li>Use the job id provided by the output to check the status of the workflow:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; curl -v http://127.0.0.1:8000/api/workflows/v1/&lt;id&gt;/status</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; #After a few minutes, the above command should return a &ldquo;succeeded&rdquo; message.</p>

<h4>Summary of the commands</h4>

<p>Here are all the commands in this section:</p>

<pre>
<code class="language-bash">su – Cromwell
cd ${GENOMICS_PATH}/cromwell
nohup java -jar -Dconfig.file=reference.conf cromwell-52-fix.jar server 2&gt;&amp;1 &gt;&gt;cromwell.log &amp;
[cromwell@frontend cromwell]$ ps aux | grep cromwell | grep server
cromwell &lt;PID&gt; &lt;…&gt; java -jar -Dconfig.file=reference.conf cromwell-52-fix.jar server
cd ${GENOMICS_PATH}/cromwell/
cat &gt;&gt; HelloWorld.wdl &lt;&lt; EOFwdl task hello {
String name command {
echo 'Hello ${name}!'
}
output {
File response = stdout()
}
runtime {
memory: "2MB" disk: "2MB"
}
}
workflow helloWorld { call hello
}
EOFwdl
echo {\"helloWorld.hello.name\":\ "World\"}" &gt;&gt; HelloWorld.json
curl -v http://127.0.0.1:8000/api/workflows/v1 -F \ workflowSource=@${GENOMICS_PATH}/cromwell/HelloWorld.wdl -F \ workflowInputs=@${GENOMICS_PATH}/cromwell/HelloWorld.json
curl -v http://127.0.0.1:8000/api/workflows/v1/&lt;id&gt;/status</code></pre>

<p>#After a few minutes, the above command should return a &ldquo;succeeded&rdquo; message.</p>

<h3><a id="3.10.6_Install_the_Burrows-Wheeler_Align"></a><a id="_bookmark92"></a><a id="_Toc92729793"></a>Installing the Burrows-Wheeler Aligner (BWA)</h3>

<p>BWA is a software package for mapping low-divergent sequences against a large reference genome, such as the human genome. In order to install the Burrows- Wheeler Aligner requires you must compile it.</p>

<ul>
<li>Login to the cromwell user account:<br />
&nbsp; &nbsp; su - cromwell</li>
<li>Create the&nbsp;<em>tools&nbsp;</em>subdirectory in the GENOMICS_PATH directory and enter that directory:<br />
&nbsp; &nbsp; mkdir ${GENOMICS_PATH}/tools</li>
<li>Download the recommended version of the BWA software package:<br />
&nbsp; &nbsp; wget https://github.com/lh3/bwa/releases/download/v0.7.17/bwa-0.7.17.tar.bz2</li>
<li>Unpack the tarball:<br />
&nbsp; &nbsp; tar -xjf bwa-0.7.17.tar.bz2</li>
<li>Compile BWA:<br />
&nbsp; &nbsp; cd bwa-0.7.17</li>
<li>Make a symlink:<br />
&nbsp; &nbsp; cd ${GENOMICS_PATH}/tools</li>
</ul>

<p><strong>Summary of the commands</strong></p>

<p>Here are all the commands in this section:</p>

<pre>
<code class="language-bash">su - cromwell
mkdir ${GENOMICS_PATH}/tools
wget https://github.com/lh3/bwa/releases/download/v0.7.17/bwa-0.7.17.tar.bz2
tar -xjf bwa-0.7.17.tar.bz2
cd bwa-0.7.17
cd ${GENOMICS_PATH}/tools</code></pre>

<h3><a id="3.10.7_Installing_the_GenomeAnalysisTool"></a><a id="_bookmark93"></a><a id="_Toc92729794"></a>Installing the Genome Analysis ToolKit (GATK)</h3>

<p>The Genome Analysis ToolKit (GATK) is the industry standard for identifying SNPs and indels in germline DNA and RNAseq data. It is a collection of command-line tools for analyzing high-throughput sequencing data with a primary focus on variant discovery. The tools can be used individually or chained together into complete workflows. The Broad Institute provides end-to-end workflows, called GATK Best Practices, tailored for specific use cases. The Intel&reg; Genomics Kernel Library with AVX-512 performance improvements is directly integrated into GATK.</p>

<ul>
<li>Navigate to the staging folder:<br />
&nbsp; &nbsp; cd ${GENOMICS_PATH}/tools</li>
<li>Download the latest version of GATK:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;wget https://github.com/broadinstitute/gatk/releases/download/4.1.9.0/gatk-4.1.9.0.zip</p>

<ul>
<li>Unpack the zip file:<br />
&nbsp; &nbsp; &nbsp;unzip gatk-4.1.9.0.zip</li>
<li>Add the PATH to the default user environment:<br />
&nbsp; &nbsp; &nbsp;echo &quot;export PATH=\${GENOMICS_PATH}/tools/gatk-4.1.9.0:\$PATH&quot; &gt;&gt; /etc/bashrc</li>
<li>Make tools symlink:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; cd ${GENOMICS_PATH}/tools</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ln -s gatk-4.1.9.0 gatk</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ln -s gatk-4.1.9.0/gatk-package-4.1.9.0-local.jar gatk-jar</p>

<p><a id="_Hlk89328560"></a><strong>Summary of the commands</strong></p>

<p>Here are all the commands in this section:</p>

<pre>
<code class="language-bash">cd ${GENOMICS_PATH}/tools
wget \
https://github.com/broadinstitute/gatk/releases/download/4.1.9.0/gatk-4.1.9.0.zip
unzip gatk-4.1.9.0.zip
echo "export PATH=\${GENOMICS_PATH}/tools/gatk-4.1.9.0:\$PATH" &gt;&gt; /etc/bashrc
cd ${GENOMICS_PATH}/tools
ln -s gatk-4.1.9.0 gatk
ln -s gatk-4.1.9.0/gatk-package-4.1.9.0-local.jar gatk-jar</code></pre>

<h3><a id="3.10.8_Picard_Install_(as_Cromwell)"></a><a id="_bookmark94"></a><a id="_Toc92729795"></a>Installing Picard</h3>

<ul>
<li>When logged on as user, Cromwell, navigate to the staging folder:<br />
&nbsp; &nbsp; cd ${GENOMICS_PATH}/tools</li>
<li>Download the recommended version of Picard<br />
&nbsp; &nbsp; wget https://github.com/broadinstitute/picard/releases/download/2.23.8/picard.jar</li>
</ul>

<p><a id="3.10.9_Install_Samtools"></a><a id="_bookmark95"></a><a id="_Hlk89328613"></a><strong>Summary of the commands</strong></p>

<p>Here are all the commands in this section:</p>

<pre>
<code class="language-bash">cd ${GENOMICS_PATH}/tools
wget https://github.com/broadinstitute/picard/releases/download/2.23.8/picard.jar</code></pre>

<h3><a id="_Toc92729796"></a>Installing Samtools</h3>

<ul>
<li>Change to the&nbsp;<em>tools&nbsp;</em>directory<br />
&nbsp; &nbsp; cd ${GENOMICS_PATH}/tools</li>
<li>Download the recommended version of the SAMtools software:<br />
&nbsp; &nbsp; wget https://github.com/samtools/samtools/releases/download/1.9/samtools-1.9.tar.bz2</li>
<li>Unpack the tarball:<br />
&nbsp; &nbsp; tar -xjf samtools-1.9.tar.bz2</li>
<li>Compile and install Samtools:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;cd samtools-1.9</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;./configure -prefix=${GENOMICS_PATH}/tools</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; make</p>

<ul>
<li>Make a symlink:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ${GENOMICS_PATH}/tools</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ln -s samtools-1.9 samtools</p>

<p><a id="_Hlk89328709"></a><strong>Summary of the commands</strong></p>

<p>Here are all the commands in this section:</p>

<pre>
<code class="language-bash">cd ${GENOMICS_PATH}/tools
wget https://github.com/samtools/samtools/releases/download/1.9/samtools-1.9.tar.bz2
tar -xjf samtools-1.9.tar.bz2
cd samtools-1.9
./configure -prefix=${GENOMICS_PATH}/tools
make
${GENOMICS_PATH}/tools
ln -s samtools-1.9 samtools</code></pre>

<h3><a id="3.10.10_Install_VerifyBamID2"></a><a id="_bookmark96"></a><a id="_Toc92729797"></a>Installing VerifyBamID2</h3>

<p>You must build the project from the source code. See the details at: https:// github.com/Griffan/VerifyBamID</p>

<ul>
<li>Navigate to the tools folder:<br />
&nbsp; &nbsp; cd ${GENOMICS_PATH}/tools</li>
<li>Create the file&nbsp;<em>build_verify.sh&nbsp;</em>and add the following contents:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; #https://github.com/broadinstitute/warp/tree/cec97750e3819fd88ba382534aaede8e05ec52df/dockers/broad/ VerifyBamId</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;cd $GENOMICS_PATH/tools</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;VERIFY_BAM_ID_COMMIT=&quot;c1cba76e979904eb69c31520a0d7f5be63c72253&quot; GIT_HASH=$VERIFY_BAM_ID_COMMIT</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;HTS_INCLUDE_DIRS=$GENOMICS_PATH/tools/samtools-1.9/htslib-1.9/ HTS_LIBRARIES=$GENOMICS_PATH/tools/samtools-1.9/htslib-1.9/libhts.a</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;wget -nc https://github.com/Griffan/VerifyBamID/archive/$GIT_HASH.zip &amp;&amp; \ unzip -o $GIT_HASH.zip &amp;&amp; \</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;cd VerifyBamID-$GIT_HASH &amp;&amp; \ mkdir build &amp;&amp; \</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;cd build &amp;&amp; \</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;echo cmake -DHTS_INCLUDE_DIRS=$HTS_INCLUDE_DIRS -DHTS_LIBRARIES=$HTS_LIBRARIES .. &amp;&amp; \</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;CC=$(which gcc) CXX=$(which g++) \</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;cmake -DHTS_INCLUDE_DIRS=$HTS_INCLUDE_DIRS -DHTS_LIBRARIES=$HTS_LIBRARIES .. &amp;&amp; \</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;make &amp;&amp; \</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;make test &amp;&amp; \ cd ../../</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;mv $GENOMICS_PATH/tools/VerifyBamID-$GIT_HASH $GENOMICS_PATH/tools/VerifyBamID &amp;&amp; \ rm -rf $GENOMICS_PATH/tools/$GIT_HASH.zip $GENOMICS_PATH/tools/VerifyBamID-$GIT_HASH</p>

<ul>
<li>Run the new script to build and install VerifyBamID2:<br />
&nbsp; &nbsp; &nbsp; sh ./build_verify.sh</li>
</ul>

<p><a id="3.10.11_20K_Throughput_Run"></a><a id="_bookmark97"></a></p>

<h2><a id="_Toc92729798"></a>Using the 20K Throughput Run to verify your configuration</h2>

<p>The 20K Sample Test provides a quick end-to-end smoke test. It runs a Broad Best Practices Workflow with a short input dataset. A single Whole Genome Sequence (WGS) can take hours to run. Using the recommended hardware, this test should complete in 30-40 minutes.</p>

<ul>
<li>Login to the cromwell user account:<br />
&nbsp; &nbsp; su - cromwell</li>
<li>Navigate to the cromwell installation folder:<br />
&nbsp; &nbsp; cd ${GENOMICS_PATH}/cromwell</li>
<li>Clone the workflow repo (as a user). Configure Git proxy settings as needed for your environment.<br />
&nbsp; &nbsp; git clone https://github.com/Intel-HLS/BIGstack.git</li>
<li>Review the values in the &quot;configure&quot; values for proper DATA and TOOL paths on your system. DATA should be a filesystem all worker nodes can reach and have at least 2TB of free space. The benchmark Step2 will download data from the Broad Institute and&nbsp; &nbsp; &nbsp; &nbsp; configure the proxy settings as setup in configure.<br />
&nbsp; &nbsp;&nbsp;vim configure</li>
<li>Configure the test based on the detail in the &quot;configure&quot; file:<br />
&nbsp; &nbsp; ./step01_Configure_20k_Throughput-run.sh</li>
<li>Download data in the DATA_PATH provided in the configure file:<br />
&nbsp; &nbsp; ./step02_Download_20k_Data_Throughput-run.sh</li>
<li>Run the Throughput benchmark:<br />
&nbsp; &nbsp; ./step03_Cromwell_Run_20k_Throughput-run.sh</li>
<li>Monitor the workflow for the Workflow status - Failed, Succeeded, Running:<br />
&nbsp; &nbsp; ./step04_Cromwell_Monitor_Single_Sample_20k_Workflow.sh</li>
<li>&nbsp;To view the final output of 20k Throughput run, execute</li>
</ul>

<p><em>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;step05_Single_Sample_20k_Workflow_Output.sh</em><br />
<br />
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;./step05_Single_Sample_20k_Workflow_Output.sh</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Total Elapsed Time for 64 workflows: &#39;X&#39; minutes: &#39;Y&#39; seconds<br />
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Average Elapsed Time for Mark Duplicates: &#39;X.YZ&#39; minutes</p>

<p><a id="3.10.12_Troubleshooting_of_the_Genomics_"></a><a id="_bookmark98"></a></p>

<h2><a id="_Toc92729799"></a>Troubleshooting the genomics software stack</h2>

<p>The following section describes steps for troubleshooting issues that may arise during the installation of the genomics software stack.</p>

<h3><a id="_Toc92729800"></a>Troubleshooting failure to log into MariaDB</h3>

<ul>
<li>Stop MariaDB:<br />
&nbsp; &nbsp; systemctl stop mariadb</li>
<li>Start a safe session and login:<br />
&nbsp; &nbsp;&nbsp;mysqld_safe --skip-grant-tables --skip-networking &amp; mysql -u root</li>
<li>Set a new password for root user or change password as desired:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;MariaDB [(none)]&gt; FLUSH PRIVILEGES;</p>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;MariaDB [(none)]&gt; SET PASSWORD FOR &#39;root&#39;@&#39;localhost&#39; = PASSWORD(&#39;password&#39;); MariaDB [(none)]&gt; EXIT;</p>

<ul>
<li>Cleanup and restart MariaDB:</li>
</ul>

<p>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;kill `cat /var/run/mariadb/mariadb.pid` systemctl start mariadb</p>

<ul>
<li>Log into the MariaDB server as the database administrator. Enter the root password when prompted and resume the setup process:<br />
<br />
&nbsp; &nbsp; &nbsp;mysql -h localhost -u root -p</li>
</ul>

<h2><a id="_Toc92729801"></a><a id="_Toc68091356"></a>Installing optional components</h2>

<p>The Intel&reg; System Configuration Utility, Save and Restore System Configuration Utility (syscfg), is a command-line utility that can be used to display or set several BIOS and management firmware settings.</p>

<h2><a id="_Toc92729802"></a>Conclusion</h2>

<p>This guide contains recommendations for tuning software used in the Intel&reg; Select Solutions for Genomics Analytics on the 3rd Generation Intel&reg; Xeon&reg; Scalable Processors Based Platform.&nbsp;<em>HPC Cluster Tuning on 3rd Generation Intel&reg; Xeon&reg; Scalable Processors&nbsp;</em>describes the hardware configuration.</p>

<h2><a id="_Toc92729803"></a><a id="_Hlk79311412"></a>Feedback</h2>

<p>We value your feedback. If you have comments (positive or negative) on this guide or are seeking something that is not part of this guide,&nbsp;<a href="https://community.intel.com/t5/Software-Tuning-Performance/bd-p/software-tuning-perf-optimization">please reach out</a>&nbsp;and let us know what you think.</p>

<h4>Notices &amp; Disclaimers</h4>

<p>Intel technologies may require enabled hardware, software or service activation.</p>

<p>No product or component can be absolutely secure.</p>

<p>Your costs and results may vary.</p>

<p>Code names are used by Intel to identify products, technologies, or services that are in development and not publicly available. These are not &quot;commercial&quot; names and not intended to function as trademarks</p>

<p>The products described may contain design defects or errors known as errata which may cause the product to deviate from published specifications. Current characterized errata are available on request.</p>

<p><strong>&copy; Intel Corporation. Intel, the Intel logo, and other Intel marks are trademarks of Intel Corporation or its subsidiaries. Other names and brands may be claimed as the property of others.</strong></p>
</p>
