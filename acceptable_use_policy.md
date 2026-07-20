# Rāpoi HPC Acceptable Use Policy
## 1. Overview
Rāpoi is a shared High-Performance Computing (HPC) resource provided to support research and teaching activities. All users are expected to use the system responsibly, efficiently, and with consideration for others.
By accessing Rāpoi, you agree to comply with this policy, as well as all applicable University ICT and data governance policies.
## 2. Acceptable Use
### 2.1 Purpose of Use
Rāpoi resources are provided for academic research and teaching purposes only.
The following are not permitted:
- Commercial or for-profit work without explicit approval
- Proxy work, including submitting jobs on behalf of others without authorisation
- Sharing accounts or credentials
### 2.2 Fair and Responsible Use
Rāpoi is a shared system, and improper usage can negatively impact other users. A fair-share scheme is enforced to facilitate equitable access across all users and research groups.
**Users must:**
- Use computational resources efficiently
- Request only the resources required for their workloads
- Follow Slurm scheduling policies and limits
- Be familiar with and regularly review the Rāpoi documentation: https://vuw-research-computing.github.io/raapoi-docs/
- Respond to queries from administrators, moderators, and other support staff in a timely manner
**Users must not:**
- Attempt to subvert fair-share or other equitable access systems
## 3. Login Node Usage Policy
### 3.1 Purpose of Login Node
Login nodes are shared access points used to interact with the cluster. They are intended for:
- Submitting jobs to the scheduler, for example `sbatch` or `srun`
- Editing scripts and files
- Compiling small or short-running code
- Small file transfers and data management
- Monitoring job status
### 3.2 Correct Usage
- All computational workloads must be submitted to compute nodes via the Slurm scheduler.
- If interactive work with significant resources is required, users must request an interactive job, for example using `srun` or `salloc`.
### 3.3 Prohibited Activities on Login Node
Login nodes are not designed for computation or intensive I/O. The following activities are strictly prohibited on login node:
- Running resource-intensive tasks such as CPU-, memory-, or I/O-heavy workloads
- Executing code, apart from very short scripts
- Launching large numbers of processes
- Processing large datasets
- Running workflows that resemble batch jobs
- Running software that involves a graphical user interface (GUI), including but not limited to development environments such as VSCode and MATLAB, and visualisation or plotting tools such as ParaView. Such tools must be run on an appropriate partition, as outlined in the Rāpoi documentation.
Such activities degrade performance for all users and may be terminated without notice.
## 4. Accounts and Security
Users are responsible for maintaining the security of their accounts:
- Do not share passwords or SSH keys
- Use secure access methods, such as SSH
- Do not allow unauthorised access to the cluster
- Report any suspected security issues immediately
- Accounts are personal and non-transferable
## 5. Data and Storage
Rāpoi storage systems are intended for active research workflows, not for long-term archival use.
Users must:
- Keep only necessary data in the home directory
- Regularly clean up temporary and intermediate files in Scratch and `/tmp`
- Maintain external backups of important data
- Avoid filling shared filesystems
- Avoid installing local copies of software that is already installed through the module system
- Data can be stored on the SoLAR system; inquiries should be directed to Digital Solutions: Research Services
Failure to manage storage responsibly may impact other users.
## 6. Job Scheduling and Resource Use
All compute work must be carried out via the Slurm scheduler.
Users must:
- Submit jobs using appropriate resource requests, including CPU, memory, and time
- Avoid oversubscription and underutilisation of resources
- Test workflows with small jobs before running large-scale workloads
- Use interactive jobs when required
Improper job submissions may be restricted or terminated to maintain system stability.
## 7. Prohibited Activities
The following are considered misuse of the system:
- Running compute jobs on the login node
- Attempting to bypass the scheduler
- Causing system instability, for example by filling storage or crashing services
- Downloading or hosting non-research-related content
- Using GUI tools on the login node, such as VSCode or MATLAB
- Any activity that violates University policies or applicable laws
## 8. Enforcement and Access Control
At the discretion of the Rāpoi administrators, a first minor breach due to inexperience or genuine misunderstanding may be addressed through guidance rather than immediate loss of access, provided it does not cause significant disruption or risk.
Violations of this policy may result in:
- Immediate termination of running processes without prior warning
- Limitations on compute resources
- Temporary or permanent suspension of access
- Reporting to supervisors or institutional authorities
- Immediate revocation of access in cases of severe misuse
## 9. Account Lifecycle
Accounts that are unused for extended periods may be disabled or removed. Users must maintain valid affiliation with the University to retain access 
## 10. Support
If you are unsure about correct usage or need help:
- Consult the Rāpoi documentation: https://vuw-research-computing.github.io/raapoi-docs/
- Contact the Rāpoi support team: Muhammad.hashmi@vuw.ac.nz
