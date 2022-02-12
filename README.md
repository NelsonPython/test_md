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

