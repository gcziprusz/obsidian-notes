
## Introduction

In a cloud continuing education course with 20–30 participants over 4–8 weeks, it’s crucial to provide each participant with an AWS environment that is free to them, while allowing the instructor oversight and control. The instructor must be able to access each participant’s AWS account (for troubleshooting/monitoring via IAM), and participants should log in easily. Additionally, each participant will use GitHub Actions for CI/CD deployments to AWS, so integration between GitHub and AWS accounts should follow best practices. Finally, the instructor wants to control or limit what AWS services can be used to prevent excessive costs or misuse. The simplest setup that meets all these requirements is preferred.

This report compares several options for setting up AWS accounts under these conditions – including **AWS Organizations with Service Control Policies (SCPs)**, **AWS IAM Identity Center (formerly AWS SSO)**, using **individual AWS Free Tier accounts**, and leveraging **third-party educational tools or credits**. We evaluate each option’s complexity, control, cost implications (especially with Free Tier), and security. A summary comparison table is provided, followed by a recommended solution with step-by-step setup instructions and best practices for account access, permission management, and GitHub Actions integration.

## Requirements Summary

Before comparing solutions, let’s summarize the key requirements for the course setup:

- **Number of participants:** ~20–30 individuals.
- **Duration:** 4–8 weeks of access.
- **Cost to participants:** Must be free (ideally leveraging AWS Free Tier or credits so participants aren’t charged).
- **Instructor access:** Instructor can log into each participant’s AWS environment (IAM access is sufficient – no shared root credentials).
- **Participant access:** Participants should have a straightforward way to log in to their AWS account (minimal setup friction).
- **CI/CD integration:** Each participant will configure GitHub Actions to deploy code to AWS – requiring secure AWS credentials or roles.
- **Service usage control:** The instructor should be able to impose limits or guardrails on what AWS services or resources participants can use (to prevent expensive resources or unauthorized services).
- **Preference for simplicity:** Among solutions that meet these needs, choose the simplest to set up and manage.

With these criteria in mind, let’s explore the possible setup options.

## Option 1: **Multiple AWS Accounts via AWS Organizations + SCPs**

**Description:** The instructor uses AWS Organizations to create and manage a **separate AWS account for each participant**. All accounts are under a single organization (with the instructor’s account as the management root). Using **Service Control Policies (SCPs)**, the instructor can enforce restrictions on these member accounts. Participants would each operate in their own AWS account (isolated from each other), and the instructor can assume an admin role in any account to troubleshoot. All accounts are consolidated under one billing owner (the instructor or institution).

**Setup & Access:** The instructor (organization admin) can create 20–30 member accounts in AWS Organizations (each requires a unique email). By default, AWS Organizations sets up a cross-account role (e.g. `OrganizationAccountAccessRole`) in each new account which the admin can assume to gain full access. Participants could be given IAM user credentials in their respective accounts, or (preferably) access via Identity Center (see Option 2). The instructor can log in to any account by assuming the role, without needing credentials from participants.

**Control:** Using SCPs at the organization or OU (organizational unit) level, the instructor can **restrict what services or actions are allowed** in participant accounts. SCPs are guardrail policies that **limit the maximum permissions** an account’s users can have ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=Q8%3A%20If%20we%20sign,Free%20Tier%20for%20each%20account)) ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=No%2C%20customers%20that%20use%20Consolidated,one%20Free%20Tier%20per%20Organization)). For example, an SCP could **deny certain high-cost services** (like blocking creation of large EC2 instance types, or disallow regions outside a specific one, etc.). This ensures participants stay within safe boundaries regardless of any IAM policies inside their account. All participants’ accounts can be placed in one OU with a tailored SCP. This gives a high level of control over resource usage and security.

**Cost:** AWS Organizations uses **consolidated billing** – meaning all usage rolls up to the management account’s bill. AWS Organizations itself is free; costs only incur from resources used in the accounts. A critical point is that **AWS Free Tier benefits are shared across an organization** when using consolidated billing. You do **not** get 20–30 separate free tiers for 20–30 accounts under one org; instead, the entire organization only qualifies for one set of Free Tier limits ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=Q8%3A%20If%20we%20sign,Free%20Tier%20for%20each%20account)) ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=No%2C%20customers%20that%20use%20Consolidated,one%20Free%20Tier%20per%20Organization)). (AWS’s policy is one Free Tier per organization, not per account, when accounts are linked ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=No%2C%20customers%20that%20use%20Consolidated,one%20Free%20Tier%20per%20Organization)).) This means if all participants actively use resources, the free tier limits (e.g. 750 hours of t2.micro EC2, 5 GB S3, etc.) could be exceeded when aggregated. The instructor should **plan for some costs** if usage won’t fit in a single free tier, or consider using credits. However, costs can still be kept low by enforcing restrictions and mindful usage. The instructor can also set up AWS Budgets or alerts on each account to monitor spending. The **“free for participants”** requirement is met since participants won’t need to enter credit cards or pay — the instructor (or institution) covers any combined bill (which ideally stays within free tier or minimal overages).

**Complexity:** Initial setup of multiple accounts via Organizations is more work than a single account, but AWS provides tools to automate account creation. With ~20–30 accounts, this is a manageable scale. Setting up SCPs requires understanding AWS policy syntax, but there are many examples available. Overall management effort is moderate: the trade-off for higher control and isolation is a bit more upfront configuration. Access management can be streamlined with Option 2 (SSO) to reduce complexity for logins.

**Security & Isolation:** Each participant having a separate AWS account provides strong isolation. Misconfigurations or security issues in one account won’t directly impact others. The instructor, as org admin, has oversight. SCPs act as an **additional security layer** – even if a participant has admin rights in their account, they cannot exceed the boundaries set by SCP (for instance, they can’t escalate privileges to break out of allowed services). This option is aligned with AWS best practices which recommend using multiple accounts for different projects or users to isolate and secure resources ([Establishing your best practice AWS environment - Amazon Web Services](https://aws.amazon.com/organizations/getting-started/best-practices/#:~:text=AWS%20resources%2C%20and%20enables%20you,practice%20that%20offers%20several%20benefits)) ([Establishing your best practice AWS environment - Amazon Web Services](https://aws.amazon.com/organizations/getting-started/best-practices/#:~:text=offers%20several%20benefits%3A)). 

**Pros:** Full isolation per participant; centralized billing and control; instructor has easy access to all accounts; can enforce service limits via SCP; participants don’t need to handle billing.  
**Cons:** Setup is more involved (multiple accounts to create/manage); **Free Tier is effectively one shared pool** for all accounts ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=No%2C%20customers%20that%20use%20Consolidated,one%20Free%20Tier%20per%20Organization)), reducing the total “free” resources available compared to separate free accounts; the instructor’s account bears the risk of any overage charges (so must monitor usage); requires managing multiple credentials or implementing SSO (below) for user logins.

## Option 2: **AWS IAM Identity Center (AWS Single Sign-On)**

**Description:** AWS IAM Identity Center (formerly AWS SSO) is a service that **centralizes user identity and access** across multiple AWS accounts. This option is not mutually exclusive with Option 1 – rather, it is a **complementary approach** to manage login credentials and permissions in a multi-account setup. Using Identity Center, the instructor can create one user profile for each participant and grant each user access to only their AWS account. Participants then use a single sign-on portal to access AWS, and the instructor can also use SSO for cross-account access.

**Setup & Access:** Identity Center is enabled at the organization level. The instructor would enable IAM Identity Center in the AWS Organization’s management account, and set up the **built-in user directory** (or integrate with an existing SAML/AD identity provider if available, but built-in is simpler for a standalone course). For each participant, the instructor creates an Identity Center user (usually with the participant’s email). Then, the instructor creates a **Permission Set** defining what permissions the user will have in the target account – for a training scenario, this could be full Admin access *within that account* (or a limited power-user policy if you want to restrict certain actions). The user is then **assigned to their specific AWS account** with that permission set ([Assign AWS account access for an IAM Identity Center user](https://docs.aws.amazon.com/singlesignon/latest/userguide/get-started-assign-account-access-user.html#:~:text=Assign%20AWS%20account%20access%20for,3%3A%20Review%20and%20Submit)). The result: each participant will receive an email invite to set up their SSO password, and afterwards can log into an AWS SSO portal URL to get one-click access to the AWS console of their account (or command-line credentials via AWS CLI integration).

**Control:** Identity Center itself doesn’t directly impose service restrictions (that’s done via IAM policies or SCPs), but it simplifies managing those policies. You can define Permission Sets that limit what a user can do in the account. For example, instead of giving students full Administrator, you might create a permission set that allows common services (EC2, S3, Lambda, etc.) but denies certain actions. However, maintaining a fine-grained permission set for students can be complex and may inadvertently block some lab activities. In practice, many instructors will give broad rights inside the sandbox account and rely on **SCPs at the org level** for coarse restrictions. Identity Center does ensure that participants only access *their* account – they won’t even see other accounts that they aren’t assigned to.

**Cost:** IAM Identity Center is free to use. It doesn’t change the billing considerations from Option 1 – accounts are still under an org with one billing. The benefit is that participants don’t create separate paid accounts, so no one needs a credit card on file except the main account. The same caution about Free Tier pooling applies (since Identity Center assumes an Organization). The combination of Organizations + SSO still has **one Free Tier overall** ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=No%2C%20customers%20that%20use%20Consolidated,one%20Free%20Tier%20per%20Organization)), so cost management needs the same attention.

**Complexity:** Identity Center adds some initial setup steps (enabling the service, creating users, etc.), but it greatly simplifies **ongoing user management**. Instead of juggling 20–30 IAM users across different accounts or sharing long-lived credentials, the instructor deals with one centralized user list. From the participants’ perspective, login is easier – a single username/password (or even federated login via Google/Microsoft if configured) gets them into the AWS console without needing access keys or account numbers. This is a modern best practice for multi-account setups and avoids emailing static passwords or keys. The extra setup is worth the improved ease-of-use.

**Security & Isolation:** SSO allows enforcing **MFA for user logins** easily (one toggle to require MFA for all Identity Center users). It also avoids the need to create IAM users in each account (reducing the surface of potential leftover credentials). Each participant gets a unique account and login, improving accountability. The instructor can instantly revoke a user’s access by disabling their SSO account or removing their assignment, without touching the member account itself. Overall, Identity Center enhances security by centralizing and simplifying credential management.

**Pros:** Easiest login experience for participants (single sign-on portal); centralized user access management; no need to distribute IAM passwords or access keys; can enforce MFA and strong password policies; integrates seamlessly with AWS Organizations (assigning users to accounts with specific permission sets) ([Assign AWS account access for an IAM Identity Center user](https://docs.aws.amazon.com/singlesignon/latest/userguide/get-started-assign-account-access-user.html#:~:text=Assign%20AWS%20account%20access%20for,3%3A%20Review%20and%20Submit)).  
**Cons:** Requires using AWS Organizations (not a standalone solution by itself); initial learning curve if unfamiliar; still requires well-defined permission sets or IAM roles for each account; does not by itself solve cost issues (still need to monitor usage). In practice, it pairs with Option 1 for a more complete solution rather than standing alone.

## Option 3: **Individual AWS Free Tier Accounts (Participant-Signed)**

**Description:** Instead of the instructor provisioning accounts, each participant could **sign up for their own AWS account**, taking advantage of AWS’s 12-month Free Tier on new accounts. The instructor would then arrange access to those accounts for oversight (e.g., participants create an IAM role or user for the instructor). This approach leverages the fact that **Free Tier benefits are granted per new account** when not consolidated ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=Free%20Tier%20benefits%20are%20not,given%20to%20each%20account%20individually)). If accounts are kept separate (no org/consolidated billing linking them), each participant potentially enjoys their own full Free Tier allotment.

**Setup & Access:** Each participant would go through AWS’s sign-up process with their email, including providing a credit card (AWS requires one even for free usage). They would then have an individual AWS account in their name. To allow instructor access, each participant would need to set up some IAM permissions for the instructor. For example, the instructor could provide a script or instructions for the participant to create an IAM role in their account that trusts the instructor’s AWS account ID. Then the instructor can assume that role to view or fix things in the participant’s account. Alternatively, participants could create an IAM user for the instructor (with appropriate permissions), but sharing a role is cleaner. This coordination is an extra step and might be technically challenging for some students, but it can be a learning exercise as well.

**Control:** Since these accounts are standalone (not under a common org), **the instructor cannot centrally enforce Service Control Policies**. Control over allowed services would rely on each account’s IAM setup. The instructor could distribute a restrictive IAM policy that participants attach to their own admin role or user, but since the participants ultimately control their accounts (they are the root users), they could bypass or alter those restrictions. Essentially, you are relying on guidance and trust that participants will not step outside the provided IAM role’s scope or spin up disallowed services. Another approach is using **AWS Organizations *after* signup** – e.g., invite each participant’s account into an organization for the duration. However, the moment you consolidate billing under an org, **multiple free tiers collapse into one** ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=No%2C%20customers%20that%20use%20Consolidated,one%20Free%20Tier%20per%20Organization)), negating the main advantage of this option. Therefore, to truly get separate free tiers, the accounts must remain unlinked in billing, which sacrifices central control.

**Cost:** The big advantage here is **maximizing Free Tier usage**. Each participant’s account has its own Free Tier limits (e.g., each can use up to 750 hours of a t2.micro, 25 GB DynamoDB, etc., without charges). This dramatically increases the total amount of AWS resources that can be used “for free” across the class. As long as participants stay within typical free-tier usage, they should incur no charges on their accounts. This keeps it “free for participants” in practice. However, there are caveats:
- Participants must be careful not to exceed free tier limits (the instructor should provide guidance on what is safe to use).
- Each participant is individually responsible for any charges if they go over – which could be a concern. The instructor might mitigate this by offering to cover any accidental overages or by supplying AWS credit codes if available.
- Some participants may be uncomfortable providing a credit card or may not have one. This could be a barrier.
  
**Complexity:** For the instructor, this approach is **simpler in terms of AWS setup** (no multi-account org to configure). However, it pushes some complexity to the participants (AWS signup and creating roles/users for access). If participants are new to AWS, walking 30 people through account creation and IAM role setup can be time-consuming. Additionally, tracking 30 separate account logins for troubleshooting (even if the instructor can assume roles, they need to switch to each account’s context manually) can be a bit of a juggle, though not too bad. On the plus side, each participant basically manages their own environment, reducing the instructor’s administrative burden of provisioning.

**Security & Isolation:** Each participant’s account is fully isolated from others (just like Option 1). From a security standpoint, this is good isolation. However, since the accounts are not centrally managed, the **instructor has limited ability to enforce security best practices**. It’s up to each participant to secure their credentials, set up MFA on their account, etc. The instructor can encourage these practices but cannot enforce them without org-level governance. There is also the risk that a participant might inadvertently do something like expose their AWS keys (since they have full root authority on their own account). Without oversight, one account could be compromised or run mining scripts, etc., and the instructor might not immediately know. (If the accounts were in an org, the instructor could see combined CloudTrail logs or get central alerts.)

**Pros:** Each participant gets full Free Tier allowances (maximizing free usage) ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=Free%20Tier%20benefits%20are%20not,given%20to%20each%20account%20individually)); very low direct cost if everyone stays in free tier; minimal initial setup for the instructor (AWS org not required); strong account isolation by default; participants gain experience creating and managing their own AWS account.  
**Cons:** Requires each participant to have a credit card and sign up, which can be a logistical hurdle; the instructor cannot easily enforce restrictions or security guardrails (relying on trust and instructions); granting instructor access to each account requires extra IAM configuration per account; monitoring costs and activity is decentralized; participants could accidentally incur charges if not careful (violating the “free” intention unless those are reimbursed or covered by separate credits).

## Option 4: **Single AWS Account with Multiple IAM Users (Shared Account)**

**Description:** The instructor could choose to use **one AWS account** for the entire class, and create an IAM user (or role) for each participant within that account. Each participant would log in as an IAM user to the same AWS account, and work in a defined sandbox space (perhaps distinguished by tags, resource naming conventions, or separate AWS IAM permission scopes). The instructor also uses the same account (either the root user or an admin IAM role) to oversee everything.

**Setup & Access:** This is initially very simple: one AWS account (could be the instructor’s existing account or a new one dedicated to the course). Create 20–30 IAM users – e.g., `student1`, `student2`, … each with a console password (and ideally MFA). Each user can be placed in an IAM group or given an IAM policy that grants access to resources. You might attempt to restrict each user to only the resources they “own” (for example, by using IAM conditions on tags or using separate IAM roles for separate project environments), but out-of-the-box, all users in the account can typically view most resources unless carefully segmented. The instructor can simply log in with their own admin credentials to see or modify anything.

**Control:** With all participants in one account, the instructor can directly manage IAM policies to control actions. For example, you could write IAM policies that **limit what services each IAM user can use**. You might create different groups with different permissions, or even tailor policies per user. It’s possible to restrict certain AWS services or enforce regions using IAM policy conditions (similar to SCP effects but within one account). However, fine-grained IAM policy management for 30 users can become complex and error-prone. Additionally, AWS has no native concept of per-user resource isolation – **all users ultimately share the same account namespace**. This means if not carefully controlled, one participant could interfere with another’s resources (e.g., terminate an EC2 instance launched by another user, accidentally modify a common S3 bucket, etc.). You could mitigate this by instructing each user to use separate resource names or tags, and using a strategy called Attribute-Based Access Control (ABAC) where IAM policies allow access only to resources tagged with the user’s ID. Setting that up is advanced and would add significant complexity.

**Cost:** All usage is consolidated in one account (one bill). This single account has one set of Free Tier limits total. With potentially 20–30 people using it, it’s **very likely to exceed Free Tier** in aggregate. For instance, the Free Tier might allow 750 hours of a t2.micro EC2 instance per month – if each of 20 participants runs an EC2 instance, that’s ~20*720=14,400 hours, far above the free allowance. Therefore, this approach almost certainly incurs charges unless the class activities are extremely minimal. The instructor (account owner) would foot the bill, which could still be kept moderate by limiting usage (and it could be within any credits or budget the course has). There’s also a risk of one participant spinning up something costly that impacts the shared bill, so strict IAM limits or vigilant monitoring are needed.

**Complexity:** Initial setup is simplest of all options (one account, create IAM users). However, maintaining order in a shared account might become complicated. Debugging issues is a challenge because all participants’ resources are intermixed. For example, listing all EC2 instances will show everyone’s instances; CloudTrail logs are combined; it may be hard to tell whose resource is whose unless naming conventions are enforced. There’s also the user experience: each IAM user would log in to the same account but just with different credentials, which is fine, but they might accidentally step on each other’s toes. If the participants are coordinated and follow strict instructions, it could work, but with 30 people exploring AWS, accidents happen. The instructor would need to invest in guardrails (complex IAM policy setups or even custom scripts to isolate environments). This starts negating the “simple” advantage of this option.

**Security & Isolation:** This option provides the **least isolation**. All participants operate in the same security boundary. A misconfiguration of an IAM policy might allow one student to access or delete another student’s resources. If any one user’s credentials are compromised, it could affect the whole environment. On the flip side, the instructor has full visibility into everything since it’s one account. But from a principle-of-least-privilege standpoint, this setup is not ideal. It’s essentially a multi-user setup in one account, which AWS supports, but requires careful multi-tenant style planning to be safe. In a training scenario, the risk of mistakes is high (that’s how people learn), so one person’s mistake could impact everyone under this model.

**Pros:** Very simple initial setup; everything is centrally in one place; the instructor already has access to all resources by default; no need for managing multiple accounts or organizations; all activity is in one set of logs.  
**Cons:** **High risk of collisions and security issues** (shared environment); difficult to limit services per user without complex IAM rules; one Free Tier shared among all (likely to incur costs); participants can inadvertently affect each other’s work; does not truly meet the requirement of isolating each participant’s AWS account (since it’s actually one account). This option is generally not recommended given the requirements for isolation and control.

## Option 5: **Third-Party Educational Platforms or Credits**

**Description:** There are also **education-focused AWS solutions** and third-party platforms that can make managing class accounts easier. These include programs like **AWS Educate Starter Accounts**, **AWS Academy**, or sandbox platforms from training providers. These options often provide **temporary AWS environments or credit vouchers** for students without requiring a personal credit card, and with an easy way for instructors to oversee or manage.

- **AWS Academy Learner Labs:** If the instructor is affiliated with an institution that participates in AWS Academy, they can create a classroom in AWS Academy. Each student is then provisioned an AWS account with a pre-loaded credit (e.g., **$100 AWS credit per student for the class duration** ([[PDF] AWS Academy Learner Lab – Educator Guide - awsstatic.com](https://d1.awsstatic.com/AWS%20Academy%20Learner%20Lab%20Educator%20Guide.pdf#:~:text=awsstatic,this%20class%2C%20students%20will))). Students access these accounts through AWS Academy’s dashboard. The accounts are real AWS accounts (with some usage restrictions and a time limit), and they automatically lock when credits are depleted or the class ends. The instructor can monitor student usage via the AWS Academy interface. This meets all requirements: free for students (credits cover usage), individual accounts for each student, instructor oversight, and even built-in cost control (since the credit is a hard limit). The downside is that AWS Academy is only available to accredited institutions and requires some lead time to set up via AWS.

- **AWS Educate Starter Accounts:** AWS Educate (a program intended for students and educators) historically offered “starter accounts” managed by a third-party platform (Vocareum). These accounts do not require a credit card and come with limited credits or service quotas. Instructors can create an AWS Educate classroom and invite students. Each student then gets credentials to a limited AWS environment. Typically, not all AWS services are available — just a subset suitable for education. This could limit what projects can be done. Also, management through the Educate web interface can be clunky, and there might be a delay in getting accounts approved. This approach is free for students and gives isolation, but it might not support the full range of AWS services or the use of GitHub Actions easily (access keys would have to be extracted, which might be against the platform rules).

- **AWS Event Engine (workshops):** AWS has an internal tool used for events/workshops that generate temporary accounts accessed via a login code (no password/credit card needed). However, this is typically only available for official AWS-led events or via AWS support and usually for short-lived workshops (hours or days), not an 8-week course, so it may not be applicable.

- **Third-Party Cloud Sandboxes:** Some online training platforms (e.g., A Cloud Guru, Qwiklabs/Skillbuilder, Cloud Academy) provide sandbox AWS environments. For example, A Cloud Guru’s “Cloud Playground” lets learners launch temporary AWS sandbox accounts under their subscription. These can be used for experiments and are free to the student (the platform covers costs). However, the instructor would need to have those subscriptions in place for all participants (which might involve licensing fees, so not exactly free unless the institution already subscribes). Additionally, integrating GitHub Actions might be tricky if the sandbox resets or credentials rotate.

**Control:** These platforms often have built-in limits (for example, AWS Academy accounts have service and region restrictions aligned with classroom needs). The instructor might not be able to fine-tune service permissions beyond what the platform allows. For instance, AWS Academy might already restrict certain resource types. The benefit is that cost is inherently controlled by credit limits.

**Cost:** For AWS Academy/Educate, AWS essentially foots the bill via provided credits (assuming the class stays within the credit). This is highly cost-effective if available. Third-party platforms require a paid subscription or license (cost to the training program, but not to participants).

**Complexity:** Using AWS Academy or Educate requires applying to those programs and using their management interfaces, which is an upfront overhead. But once set up, provisioning accounts is streamlined (a few clicks to create a class and get accounts). For third-party platforms, complexity is in managing another system and ensuring it meets the course needs.

**Security & Isolation:** Each student still gets an isolated account (or an isolated slice of an account) in these scenarios, so isolation is strong. The platforms often have oversight tools for instructors, but you may be abstracted away from direct AWS access (in AWS Educate Starter accounts, for example, instructors view usage through the Educate portal rather than directly in the AWS console). 

**Pros:** Potentially the easiest for students (no credit cards, no manual signup); free credits cover usage; built-in time and usage limits prevent cost overruns; good isolation per student; reduces burden on instructor to manually configure accounts.  
**Cons:** Availability is limited (must be an educator in AWS Academy or AWS Educate program, or have subscriptions to a training platform); not as flexible with service selection (some services may be disallowed by the program); less direct control for the instructor on fine details of permissions; integration with GitHub Actions might require extra steps (e.g., extracting AWS credentials from the platform to use in GitHub, which might be discouraged or complicated).

## Comparison of Options

The following table summarizes the key trade-offs of the above options:

| **Option**                                   | **Complexity** (Setup & Management) | **Control & Restrictions**        | **Cost Considerations**                   | **Security & Isolation**             |
|----------------------------------------------|-------------------------------------|-----------------------------------|-------------------------------------------|--------------------------------------|
| **AWS Org + SCPs** <br>(*multiple accounts, instructor-managed*) | Moderate – create 20–30 accounts, configure org. Some automation or scripting may help. | High – Can enforce **SCP guardrails** on all accounts (deny certain services, regions, etc.). Each account is separate. | One consolidated bill. **One Free Tier for all** ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=No%2C%20customers%20that%20use%20Consolidated,one%20Free%20Tier%20per%20Organization)), so free usage is shared. Risk of charges if combined usage > Free Tier, so monitor and possibly use credits. | Strong isolation (separate accounts). Instructor has full oversight via org. Security high with SCPs limiting max permissions. |
| **AWS Org + Identity Center** <br>(*multi-account with SSO*) | Moderately High – initial org setup + enable SSO + create users. After setup, management is easy. | High – Same SCP capability as above. Also can manage fine-grained IAM permission sets per account/user if needed. | Same as above (consolidated billing, one Free Tier pool). SSO itself is free. | Strong isolation (accounts). Improved security through centralized IAM (SSO) and easier MFA/enforcement. No shared credentials. |
| **Individual Free Tier Accounts** <br>(*each participant signs up separately*) | Low for instructor (no org to set up), but Moderate overall – must coordinate 20–30 signups & set up roles for access. | Low/Moderate – No central SCPs. Instructor can only suggest or manually implement IAM limits in each account. Participants as root can override settings. Relies on trust and guidelines. | **Multiple Free Tiers (one per account)** ([amazon web services - Lambda pricing limited to account or organization? - Stack Overflow](https://stackoverflow.com/questions/63240619/lambda-pricing-limited-to-account-or-organization#:~:text=Free%20Tier%20benefits%20are%20not,given%20to%20each%20account%20individually)) – very cost-efficient for free usage. Participants manage their own billing (no charges if careful). Instructor not charged directly. | Strong isolation (separate accounts). However, less oversight – each account is managed by the participant. Security depends on each user’s practices (instructor has limited enforcement). |
| **Single Shared Account** <br>(*IAM users for each participant*) | Very Low initial setup – one account, create users. High ongoing complexity to avoid conflicts. | Moderate – All control via IAM policies. Can restrict actions, but hard to isolate users from each other’s resources. No SCPs (single account). | One bill, one Free Tier for all. Free usage likely exceeded quickly if everyone is active, so expect charges. Instructor pays all costs (could use a budget/limit). | Weak isolation – all resources intermixed. Security risk if one user misconfigures something. Instructor can see everything, but so can determined users if not strictly restricted. |
| **Third-Party / AWS Educate** <br>(*AWS Academy, Educate Starter, etc.*) | Moderate – Need to set up via external program (some paperwork). Once done, account provisioning is easy. | Moderate – Program may impose some limits (which helps). Custom control is limited to what the platform allows. | Very low cost to instructor or students if using AWS-provided credits (Academy/Educate). Third-party platforms may require a subscription fee. | Strong isolation (separate accounts or sandboxes). Security is managed by the platform’s guidelines. Instructor has oversight through the platform. |

From the comparison, the **AWS Organizations-based approach (multi-account)** stands out for giving the instructor strong control and security, aligning with best practices for isolation. The main drawback is the sharing of free tier limits, which could lead to minor costs if many resources are used. The **individual account approach** maximizes free tier usage but sacrifices control and convenience. The **single account approach** is not ideal for this use case due to isolation and cost issues. **Third-party/education programs** are excellent if available, but assuming we want to stick to regular AWS setups, we’ll focus on AWS-native solutions.

## Recommended Solution: AWS Organizations with IAM Identity Center (SSO) and SCP Guardrails

Given the requirements and trade-offs, **the recommended setup is to use AWS Organizations to provision individual AWS accounts for each participant, and use AWS IAM Identity Center (SSO) for easy login management and instructor access**, combined with Service Control Policies to restrict usage. This approach meets all the requirements with a reasonable level of simplicity and offers a good balance of control, security, and cost management:

- **Free for participants:** Participants won’t need to provide credit cards or pay, and the instructor can ensure resources stay within Free Tier or credits to avoid charges.
- **Instructor access:** The instructor (organization admin) can assume a role in any account or use SSO to jump into any participant account for support.
- **Easy login:** Participants get an email invite and can log in via a single sign-on link – no complex account setup on their part.
- **GitHub Actions integration:** Each account can be set up with the necessary IAM role for GitHub Actions, and the instructor can standardize this across accounts.
- **Control over services:** Using SCPs at the org level, the instructor can disable costly services or actions globally (e.g., prevent usage of resources that aren’t covered by free tier or needed for the course).
- **Simplicity:** While multi-account has setup overhead, it’s a one-time effort. After that, management is straightforward. The organizational structure actually simplifies oversight (all accounts in one place) and cleanup after the course (the instructor can quickly suspend or delete accounts).

Below are **setup instructions and best practices** for implementing this recommended solution:

### 1. Set Up an AWS Organization and Create Member Accounts

1. **Prepare the root (management) account:** The instructor should use or create an AWS account that will serve as the master of the organization. Enable MFA on it and ensure a valid payment method is set (this account will receive the consolidated bill for all participant accounts).

2. **Create an Organization:** In the AWS Management Console, go to **AWS Organizations** and follow the steps to create a new organization (if not already part of one). This will make your account the management account.

3. **Create Organizational Units (OUs):** For cleanliness, create an OU named “Students” or “ClassAccounts”. You can put all participant accounts in this OU. (OUs let you apply SCPs more easily to groups of accounts.)

4. **Create member accounts:** For each participant, create a new AWS account under the organization:
   - In AWS Organizations, choose **Accounts > Add account > Create account**.
   - Provide a unique email for the new account and an account name (e.g., the participant’s name or “ClassAccount<number>”). You can use email aliases if you own a domain – e.g., `instructor+student1@mydomain.com` for student1, `+student2` for student2, etc., which directs emails to the instructor’s inbox for now. *(Later, you can change the email on each account to the participant’s email if desired, but it’s not necessary since they will use SSO to log in.)*
   - AWS will send a verification email for each account created. Since you control the email (via alias), verify each one.
   - **Note:** Account creation can also be automated with AWS CLI/SDK if doing it manually is too slow, but for ~20 accounts, manual with copy-paste of an email alias is workable.

5. **Organization Account Access Role:** By default, AWS Organizations creates a role in each new account called `OrganizationAccountAccessRole` (assuming you left the default setting). The management account (your account) can assume this role to gain AdministratorAccess in the member account. This will be useful for initial setup in those accounts.

6. **Initial account setup:** It’s a good practice to log in to each new account at least once (via the assume role) to do minimal setup:
   - Enable IAM access to billing data (so the management account can see each account’s usage).
   - Turn on AWS CloudTrail and AWS Config in each account (or use an **Organization Trail** from the master to cover all accounts). This ensures all actions are logged, which is important for security audits. AWS Organizations can be used to create a single CloudTrail that logs all accounts.
   - Check that the accounts have no quotas or support plan issues. (By default, they’re on basic support which is fine.)
   - You could also use the master account to programmatically install a budget or alarm on each account (via AWS Budgets) to email the instructor if any account’s charges exceed, say, $5. This isn’t strictly necessary but is a safety net.

### 2. Enable AWS IAM Identity Center (SSO) for User Logins

1. **Enable IAM Identity Center:** In the AWS console of the management account, navigate to **IAM Identity Center (formerly AWS SSO)** service. If it’s the first time, click “Enable IAM Identity Center”. This will set up a default identity directory for your organization.

2. **Add Users:** In IAM Identity Center, go to **Users** and create a user for each participant. You just need a username, email, and optionally their name. Use each participant’s real email here if you want them to receive the login details directly. (If you don’t have their emails or want to control the invites, you could use dummy emails and later share credentials, but using their email is easier.) Upon creation, an email will be sent to them with a link to set up their SSO account (they will set their own password). You can also require MFA for these users in the settings.

3. **Create a Permission Set:** Decide what level of access participants should have in *their own accounts*. For a training environment, giving them Administrator access within their account is simplest (so they can do anything not explicitly blocked by SCPs). If you prefer to limit their powers further, you can create a custom Permission Set now:
   - For example, a Permission Set called “StudentAdmin” that is based on the AWS managed policy AdministratorAccess. You could start with AdministratorAccess and then attach an inline policy to remove certain permissions if needed. However, since we will use SCPs to prevent truly disallowed actions, it’s usually fine to just give AdministratorAccess at the account level, so they have full use of the allowed services.
   - Another approach: use “PowerUserAccess” (which is admin without the ability to manage IAM users and some account settings) as the permission set, which prevents them from creating more IAM users or roles in their account. But if part of the course is learning IAM or creating deployment roles, they might need IAM permissions too. We assume participants will be admin of their own account for flexibility.

4. **Assign Users to Accounts:** Now, for each Identity Center user (participant), assign them to their corresponding AWS account:
   - In IAM Identity Center, go to **AWS Account Access**. You’ll see your org accounts listed. For each student account, choose **Assign User** and select the appropriate user and the Permission Set (e.g., AdministratorAccess or your custom set). Do this for each account-user pair. This effectively creates an IAM role in the target account and links it to the user’s SSO profile.
   - Verify that each user is only assigned to *their* account (you don’t want to give one student access to another’s account).

5. **Instructor SSO access (optional):** The instructor can also use Identity Center for their own access to each account. One way is to create an Identity Center user for yourself and assign it to all student accounts with an Admin permission set. But since you already have the ability to assume the `OrganizationAccountAccessRole` from the management account, this is optional. However, using SSO could allow you to have a nice single sign-on view of all accounts as well. This is up to preference. (You could also integrate your corporate SSO if this is in a company setting, but assume not here.)

6. **Participant Login Process:** The participants will get an email to finalize their SSO account setup (they’ll set a password, and possibly MFA if you required it). They will use the provided link (which goes to your Identity Center user portal, something like `https://<your-alias>.awsapps.com/start`). Make sure they bookmark this URL. Through that portal, after logging in, they will see a tile for the AWS account they have access to (it will show the account name you set, e.g., their name or “ClassAccount1”). By clicking that, they get into the AWS Management Console of their account without needing to enter an AWS username/password. They can also use the **AWS CLI v2** which has SSO support to get command-line credentials if needed ([Configure user access with the default IAM Identity Center directory](https://docs.aws.amazon.com/singlesignon/latest/userguide/quick-start-default-idc.html#:~:text=Configure%20user%20access%20with%20the,the%20next%20page%2C%20enter)) ([How do I use the IAM Identity Center and the AWS access portal?](https://repost.aws/knowledge-center/user-portal-for-aws-sso#:~:text=How%20do%20I%20use%20the,%C2%B7%20Use%20the%20AWS)).

**Best Practice:** Communicate clearly with participants on how to use the SSO login and emphasize security (e.g., if not enforcing MFA via SSO, ask them to still enable MFA on their IAM Identity Center account or at least on the root of their AWS account if they ever use it). The nice thing is they technically won’t even need to know their account root password at all, operating solely through SSO.

### 3. Apply Service Control Policies (Guardrails)

To prevent misuse or accidental high charges, set up one or more **Service Control Policies** in the organization:

1. **Identify restricted services or actions:** Decide what you want to block or limit. Common restrictions for a classroom environment:
   - Deny usage of regions other than the one you’re using for the course (to avoid spreading resources and possibly higher costs in certain regions).
   - Deny launching of very large EC2 instance types, or any GPU instances, etc. You might allow t2.micro, t3.micro, but deny anything beyond, for example. (This can be done by an SCP checking the instance type as a condition on ec2:InstanceType.)
   - Deny any use of certain expensive services entirely (e.g., Amazon SageMaker, Redshift, etc., if they are not needed for the course).
   - Deny turning off encryption or modifying CloudTrail logs (security guardrail).
   - Deny creating additional AWS accounts (to prevent students from spawning sub-accounts).
   - Possibly deny IAM actions that could give them access outside their account (though if they only have their account, this is less an issue).

2. **Create SCPs:** In AWS Organizations, go to **Policies > Service Control Policies** and create a new SCP JSON policy. For example, an SCP might look like:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "DenyRestrictedRegions",
         "Effect": "Deny",
         "Action": "*",
         "Resource": "*",
         "Condition": {
           "StringNotEquals": {
             "aws:RequestedRegion": "us-east-1"
           }
         }
       },
       {
         "Sid": "DenyExpensiveInstances",
         "Effect": "Deny",
         "Action": "ec2:RunInstances",
         "Resource": "arn:aws:ec2:*:*:instance/*",
         "Condition": {
           "ForAnyValue:StringEquals": {
             "ec2:InstanceType": [
               "t2.micro",
               "t3.micro"
             ]
           }
         }
       },
       {
         "Sid": "DenyUnusedServices",
         "Effect": "Deny",
         "Action": [
            "aws-marketplace:*",
            "aws-dataexchange:*",
            "redshift:*",
            "sagemaker:*",
            "monitron:*"
            // ... (list services to block)
         ],
         "Resource": "*"
       }
     ]
   }
   ```
   The above is an illustrative sketch (note: the instance type condition should actually use `StringNotEquals` with a list of allowed types or a `Null` check logic because as written it denies if the type equals micro, which is opposite of intended – you’d want to deny if the instance type is *not* micro. Be careful to get the logic right in practice!). The idea is: one statement denies any action outside us-east-1, another denies launching instances that are not t2.micro or t3.micro (hence only micro allowed), and another denies entire services by not allowing any action of those services. You can find templates for SCPs in AWS documentation.

3. **Attach SCP to OU:** Attach your SCP(s) to the “Students” OU that contains all the student accounts. By default, AWS Organizations has an implicit allow (meaning if you don’t create any SCP, everything is allowed). Once you attach a Deny SCP, it will override any permissions in the accounts. Test this with one account first if possible – e.g., try to launch a disallowed instance type in a student account and ensure it fails.

4. **Leave room for needed services:** Make sure your SCP isn’t too restrictive, or it might impede the class. If uncertain, you might start with lighter restrictions (maybe just block a few obvious dangerous services) and add more if needed. Monitor usage in the first week to see if anyone is doing something unexpected.

**Best Practice:** Use AWS **Billing alerts** at the org or account level as a backstop. For example, set a billing alarm to email the instructor if any student account incurs more than $1 of charges in a day, or if the overall bill exceeds a threshold. This will catch any scenario where, say, an SCP missed something and a student left a resource running.

### 4. Set Up GitHub Actions Deployment for Each Account

Each participant will deploy code from GitHub Actions to their AWS account. The recommended, secure way to do this is to use **OpenID Connect (OIDC) federation** between GitHub and AWS, rather than long-lived AWS keys. GitHub Actions can request a short-lived token from AWS if AWS trusts GitHub’s OIDC identity for that repository ([Configuring OpenID Connect in Amazon Web Services - GitHub Docs](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#:~:text=OpenID%20Connect%20,lived%20GitHub%20secrets)). Here’s how to set it up for each account (the instructor can provide a guide or even automate some of this via scripts run as the org admin):

1. **Create an OIDC IAM Provider in AWS:** In each student’s AWS account, an admin (the student or the instructor assuming role) should create an IAM Identity Provider of type “OpenID Connect” with the following:
   - Provider URL: `https://token.actions.githubusercontent.com`
   - Audience: `sts.amazonaws.com`
   
   This establishes trust of GitHub’s OIDC tokens for the account.

2. **Create an IAM Role for GitHub Actions:** Still in the student’s AWS account, create an IAM role that will be assumed by GitHub Actions workflows. Key settings:
   - **Trust policy:** Configure the role’s trust to allow the OIDC provider (GitHub) to assume it. The AWS docs and GitHub docs provide the exact trust policy syntax ([Configuring OpenID Connect in Amazon Web Services - GitHub Docs](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#:~:text=Adding%20the%20identity%20provider%20to,AWS)) ([Configuring OpenID Connect in Amazon Web Services - GitHub Docs](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#:~:text=To%20configure%20the%20role%20and,for%20GitHub%20OIDC%20identity%20provider)). You’ll have conditions to ensure only a specific repository (and possibly specific branch or workflow) can assume the role. For example:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Principal": {
             "Federated": "arn:aws:iam::<AccountID>:oidc-provider/token.actions.githubusercontent.com"
           },
           "Action": "sts:AssumeRoleWithWebIdentity",
           "Condition": {
             "StringEquals": {
               "token.actions.githubusercontent.com:sub": "repo:<GitHubUsername>/<RepoName>:ref:refs/heads/main"
             }
           }
         }
       ]
     }
     ```
     This example condition says: allow the token if it’s coming from GitHub repository `GitHubUsername/RepoName` on the `main` branch. The participant would replace these with their GitHub account/repo and branch for the course project.

   - **Permissions policy:** Attach a policy to this role that defines what the GitHub Actions workflow is allowed to do in AWS. This should be **narrowly scoped to the resources the student will deploy**. For example, if they are deploying a web application, the role might allow creation of certain EC2 resources, an S3 bucket, Lambda functions, etc. Keep it least-privilege – if they only need to deploy to a specific S3 bucket or CloudFormation stack, scope the permissions to that. In a general sense for a course, you might allow a broad range of actions on common services but still avoid anything outside the project’s context.

3. **Configure GitHub Actions Workflow:** On the GitHub side, the participant needs to update their Actions workflow YAML to use OIDC to assume the AWS role. GitHub has an official action **`aws-actions/configure-aws-credentials`** that handles this. For example, a step in the workflow can be:
   ```yaml
   - name: Configure AWS credentials
     uses: aws-actions/configure-aws-credentials@v2
     with:
       role-to-assume: arn:aws:iam::123456789012:role/GHADeploymentRole
       aws-region: us-east-1
   ```
   This will perform the OIDC token exchange behind the scenes and assume the specified role in the AWS account, providing temporary credentials for subsequent steps (like AWS CLI calls, CDK deployments, etc.). No static secrets needed! As GitHub’s docs highlight, OIDC allows access to AWS “**without needing to store AWS credentials as long-lived GitHub secrets**” ([Configuring OpenID Connect in Amazon Web Services - GitHub Docs](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#:~:text=OpenID%20Connect%20,lived%20GitHub%20secrets)), improving security.

4. **Alternate simpler approach (if not using OIDC):** If the above is too complex for the course level, the fallback is to create an IAM User with access keys for CI/CD. For each student account, create an IAM user (or use the root user’s access keys, but better not) with a policy for deployment. Generate an Access Key ID and Secret Key, and have the participant add those as encrypted secrets in their GitHub repository. Then the Actions workflow can use those credentials. **However, this method is less secure** (keys can leak or be misused) and doesn’t scale as nicely. OIDC is the modern best practice for GitHub to AWS federation and avoids managing secrets ([Configuring OpenID Connect in Amazon Web Services - GitHub Docs](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#:~:text=OpenID%20Connect%20,lived%20GitHub%20secrets)).

5. **Provide instructions or scripts:** The instructor can provide a CloudFormation template or script that the students run to set up their IAM OIDC provider and role to simplify this process. Or do it for them by assuming into each account, but that might be tedious. It might be a good learning exercise for participants to set this up themselves with guidance.

**Best Practices for CI/CD:** Ensure the IAM roles for GitHub have just the permissions needed. Encourage students to use repositories that are private (or at least ensure no one malicious can run code on their repo to abuse the role – the trust policy tying to branch helps mitigate that). Also, remind them to rotate/remove those roles after the course or if a repository is deleted, to avoid any lingering access.

### 5. Testing and Ongoing Management

- **Test accounts and access:** Before the course starts, test that each participant’s SSO login works (you can send yourself an invite or use a test user). Log in as a student via SSO to verify the environment. Test the SCPs by attempting a restricted action to ensure it’s blocked. Also test the GitHub Actions deployment (maybe have one sample repo connected to one of the accounts to ensure the pipeline assumes the role correctly).

- **During the course:** Monitor usage periodically. AWS CloudTrail (and AWS CloudWatch metrics/billing) in the master account can help watch that no one spins something up outside the expected bounds. Because each student is fairly isolated, troubleshooting is mostly per-account. If a student runs into an issue, the instructor can assume into their account via the organization role or via the SSO portal as an admin and then investigate directly in their AWS console.

- **Support and permissions tweaks:** Be prepared to adjust IAM policies or SCPs if you find they are too restrictive and blocking legitimate work. For example, if an SCP was too broad and is stopping a needed API call, you might need to update it. It’s easy to update and the change propagates to accounts near instantly.

- **Cost management:** Keep an eye on the billing dashboard for the organization. If the course is within free tier allowances mostly, you might see $0 or a few dollars. If something starts incurring costs (maybe someone left an EC2 on 24/7), you can decide if it’s acceptable or if you should reach out to that participant to shut it down (or enforce a limit via SCP or by stopping the resource yourself). Since the course is only weeks long, even if free tier is exceeded, the charges might be small (for instance, an extra small EC2 instance for a month might be ~$8). You can use AWS Credits (from AWS Educate or other sources) if available to offset this.

- **Course completion:** After the course, you have some options:
  - If participants want to keep their AWS account, you could **remove their account from the organization** and hand it over. However, remember that all these accounts currently bill to your master. To let a participant take ownership, they would need to separate the account and add their own billing method. This is a bit involved (would require AWS support to fully transfer account ownership if they don’t have root credentials). Alternatively, since you created these accounts with your email alias, you actually hold the root user access. You might not want to give out those accounts permanently.
  - Easier: after course, either **suspend or delete the accounts**. You can suspend by just not using them (perhaps remove their SSO access so they can’t do more). AWS Organizations allows you to **close** accounts, which will eventually delete them after some period. Make sure any important data is saved by students (S3 downloads, etc.) before deletion.
  - If some participants are from within your company and the accounts were meant to transition to them, you can change the emails and send them the root credential reset info, then remove from org.
  - Also, remove any IAM resources that are no longer needed (like the GitHub Actions roles if leaving accounts open).

### 6. Additional Best Practices and Tips

- **Tagging:** Encourage students to tag their resources with their name or a project identifier. Since each is in their own account, tagging is less critical for separation (they have the account boundary), but it’s good practice for cost attribution and cleanup. If the instructor is looking at a consolidated bill, resource tags (with cost allocation tags enabled) can show which account or project things belong to.

- **Limit AWS Free Tier Scope:** Remind participants to stick to the services and usage patterns that align with free tier when possible. For example, use t2.micro EC2 instances (which are free tier eligible) on Amazon Linux, use DynamoDB with <= 25 RCUs/WCUs (free), etc. The SCP can enforce some of this, but user education helps too.

- **Communication:** Clearly communicate that although the environments are free to use, they are *not play-grounds for Bitcoin mining or running production loads*. Any unexpected usage will be noticed and potentially shut down. This usually isn’t an issue in an educational context, but it’s worth stating.

- **Leverage AWS Free Tier features:** For example, if students need a relational database, consider using Amazon Aurora Serverless v2 in the free tier or an RDS db.t2.micro (which has some free hours). If they need to deploy containers, use Fargate with small task sizes (some free CPU/mem each month). Wherever possible, design course projects to stay within free allowances.

- **Consider AWS Budgets for each account:** You could pre-create a Budget in each account that triggers an email to the participant and/or instructor at $0 (just to test) and $10 (to warn of overage). Participants can also view their own billing dashboard (you might have to give them permissions if they use IAM user login, but via SSO with admin role they can see billing by enabling it). Keeping everyone aware of costs is educational in itself.

- **Documentation for students:** Provide a short guide or cheat-sheet for how to log in via SSO, how to configure their GitHub Actions (with an example snippet), and what to do if something fails (e.g., reach out to instructor with the error message, etc.). If an SCP denies an action, the error can mention “blocked by organization policy”, so tell students if they see that, it likely means they attempted something outside allowed range (like a service that’s off-limits).

By following the above plan, you will have a setup where each participant has an AWS account (meeting the **isolation**, **instructor access**, and **easy login** requirements), no participant has to pay anything (the instructor can cover any minor costs, staying mostly in **Free Tier**), and the instructor retains **control** over the environment (through SCPs and central management). The integration with **GitHub Actions** is handled in a secure way using OIDC, avoiding long-lived credentials ([Configuring OpenID Connect in Amazon Web Services - GitHub Docs](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#:~:text=OpenID%20Connect%20,lived%20GitHub%20secrets)) and following cloud best practices.

This approach is scalable, secure, and relatively simple to operate after the initial configuration. It aligns well with AWS best practices for multi-account setups (using separate accounts as a boundary and central governance) while tailoring to the needs of a classroom. Each participant gets the full experience of deploying to their “own AWS environment” without the hassle of billing, and the instructor can confidently oversee the class’s cloud usage. 

