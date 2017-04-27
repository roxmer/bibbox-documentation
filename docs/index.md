# Introduction
For full documentation visit <http://b3africa.org>.

## What is BiBBox / eB3kit?


* [The eB3Kit - From a user’s perspective](#users-perspective)

    1. [An informatics platform](#informatics-platform)
    
    2. [An ethical and legal framework](#ethical-legal-framework)
    
    3. [A training component](#training-component)
    
* [BiBBoX is the eB3Kit’s operating system (OS)](#operating-system)

* [More on the eB3Kit – From the IT perspective](#more)


## <a name="users-perspective"></a>The eB3Kit - From a user’s perspective

Learn the basics of the eB3Kit with this interactive infographic: [5Ws about the eB3Kit](http://www.b3africa.org/?page_id=1494). 

#### The eB3Kit consists of 3 main components:
1. An informatics platform
2. An ethical and legal framework
3. A training component.

It comes with flexible hardware options and may be installed either on **servers or computers.**
 
The eB3Kit is designed to function both as **a stand-alone instance** and as part of a **network.** Once installed, its tools can operate **with or without an internet connection.**

#### <a name="informatics-platform"></a>1. An informatics platform

The eB3Kit informatics platform is a set of **open-source** informatics **tools and software** that may be useful to biobank and research professionals.
 
Those tools are integrated through and **ID management system** which enables the **tracking of sample data throughout the research cycle** while also **respecting an ethical and legal framework for sample** and data sharing.
 
The eB3Kit applications/ tools cover the **whole research cycle:**

* **Biobanking** 
    * Mobile data collection tools
    * Selected Laboratory Information Management Systems (LIMS), tailored to biobanks’ need
    * Biobank monitoring tools
    * Data collection and analysis tool specific to certain study areas (e.g. for collecting and analyzing phenotypic information for patients with genetic disorders).
* **Biomedical Research – Experimentation**
    * Tool for building and managing online surveys and databases
    * Experiment Management System (EMS) for omics experiments
* **Biomedical Research – Analysis** 
    * Tools for data analysis
    * Bioinformatics tools
* **Biomedical Research/ Biobanking – Data Management & Sharing**

The eB3Kit informatics platform also includes applications that may be useful for the management of research projects as well as applications to facilitate the management of the informatics platform.

Click on the image below and watch this extract from the **eB3Kit presentation (2:23)** for a quick overview of how the eB3Kit may be use throughout the research cycle:

![System Overview](images/introduction/system_overview.png "System Overview")

(Video will follow)
 
#### <a name="ethical-legal-framework"></a>2. An ethical and legal framework

The eB3Kit includes an ethical and legal framework for the storage, analysis, and sharing of biobanking and research data. It is being designed to **ensure both protection of individuals and scientific freedom,** with the main goal to **enable national, regional and international research collaborations** by eB3Kit users from highly different ethical and legal settings.

The ethical and legal framework is built upon and **aligns with the “Ethics and Governance Framework for Best Practices in Genomic Research and Biobanking in Africa”** elaborated in the frame of the H3Africa Working Group on Ethics’ activities.
  
At the backbone of the ethical and legal framework are **two minimum requirements** that would have to be fulfilled by eB3Kit users to allow sharing of data or sample for research: informed consent for all data and ethical approval.
 
**Informed consent for all data**
 
A freely given consent by a person to use his or her biological samples and associated data for health care or research purposes.

The person, or where applicable, his or her legal representative, should be given appropriate information beforehand as to the purpose of the research, the nature of the procedures to be conducted, as well as on its risks and consequences. The consent may be withdrawn freely at any time.

Limited and proportionate exceptions might be applicable where appropriate, after approval by an Ethics Review Board. Ethics Review Board could further set requirements in regards to form and content of the information given and the drafting of the consent form.

**Ethical approval**
 
Ethical approval is a decision or opinion from an ethics review board that has been authorized to independently review and approve research studies from an ethical point of view. The board consists of competent experts as well as lay persons or religious representatives, as appropriate.

Equivalent to an ethical approval is a decision or opinion issued by a Data Protection Authority, if in accordance with the applicable law.

**Agreements**

In addition to the minimum requirements, agreements between eB3Kit users will be used to comply with other ethics rules and regulations specific to the users’ setting.

Click on the image below and watch this video presentation on the **Ethical and Legal Framework (12:45):**

(Video will follow)


**Model Data Management Policy**

The ethical and legal framework is implement in the eB3Kit through a Model Data Management Policy. This policy defines eB3Kit’s actors, roles, access controls and audit of relevant processes. It also provides standard forms.

(Link will follow)


#### <a name="training-component"></a>3. A training component

In order to ensure an efficient use of the informatics platform and ethical and legal framework, the eB3Kit includes a training component. This component consists of:

* Documentation on the use of the eB3Kit itself
* Learning resources on prerequisite skills for an optimal use of the eB3Kit: video presentations from past B3Africa webinars, case studies and frequently asked questions (FAQs) 
* Links to existing resources


## <a name="operating-system"></a>BiBBoX is the eB3Kit’s operating system (OS)

It allows developers to bring together all services and tools provided with the eB3Kit under a unified user interface. It also enables users to build eB3Kits tailored to their needs, before installing them in their respective center.

Tools available to be uploaded to an eB3Kit are presented in the BiBBox “Applications Store”. Users can therefore chose which tools (“applications”) are relevant to its institution. 


## <a name="more"></a>More on the eB3Kit – From the IT perspective

All services running in the eB3Kit are “dockerized” in a way that enables easy addition/ removal of tools and services to the platform.

All files uploaded or produced by users are controlled using iRODS, which allows the storage and sharing of large data sets with the ability to tag/add meta data.
 
BiBBOox VM is hosted on a docker engine which allows data to be transferred without internet connection (using a hard disk).
