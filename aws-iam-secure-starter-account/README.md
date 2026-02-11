# Instructions

I want to do one short IAM project for aws. I want to get in, build something I can learn and do for myself (understood I have like zero understanding of aws and I’ll be doing this with a friend who is not in IT) 

Create me a project to do

Here’s a **short, beginner-friendly AWS IAM project** that teaches you the *one IAM skill you’ll use forever*: **least privilege + MFA + “break-glass” admin**.

## Project: “Secure AWS Starter Account” (IAM Foundations)

**Goal:** Build a safe AWS account setup you can reuse in real life:

- one daily user (not admin) - **completed**
- MFA enforced - **completed**
- one emergency admin (“break-glass”)
- a simple **read-only** role you can switch into **completed** (Dont forget to toggle between all filters to find your policy)

### What you’ll build (in ~45–75 min)

1. **Enable MFA on the root account** (critical) - **completed**
2. **Create an Admin group** + attach AdministratorAccess - **completed**
3. **Create a “break-glass” admin user** (only for emergencies)
4. **Create a “daily” user** with *no permanent admin rights*
5. **Require MFA for console access** (so stolen passwords aren’t enough) **complete but Review**
6. **Create a ReadOnly role** and practice **switching roles** in the console - **completed**

---

## Step-by-step checklist (exact actions)

### 0) Prep (5 min)

- Sign into AWS Console as **root** (only for initial setup).
- Go to **IAM**.

### 1) Root account: turn on MFA (5 min)

- AWS Console → search **“IAM”** → on the IAM dashboard, find **Root user** security recommendations
- Enable **MFA** (use an authenticator app).

✅ Outcome: root is protected; you won’t use it day-to-day.

---

### 2) Create an Admin group (5 min)

- IAM → **User groups** → **Create group**
- Name: `Admins`
- Attach permissions policy: **AdministratorAccess**
- Create group.

✅ Outcome: a clean way to grant admin later without attaching policies to users.

---

### 3) Create “break-glass” admin user (10 min)

- IAM → **Users** → **Create user**
- Name: `breakglass-admin` Password - BlackHawk **complete**
- Select: **Provide user access to the AWS Management Console completed**
- Set a strong password + require password reset at first sign-in (optional) **complete**
- Add user to group: `Admins` **complete**
- Create user **complete**
- Then go into that user → **Security credentials** → **Assign MFA device**. **Completed**

✅ Outcome: you have a protected emergency admin with MFA.

**Rule:** Only use this user if you lock yourself out or need emergency fixes.

---

### 4) Create your daily user (10 min)

- IAM → Users → Create user
- Name: `user-daily` (or whatever) **completed**
- Console access: yes **completed**
- **DO NOT** add to Admins **completed**
- Create user **completed**
- Assign MFA to this user too. **Completed**

✅ Outcome: your normal login exists and is not admin.

---

### 5) Enforce MFA for IAM users (15 min)

We’ll attach a policy that **blocks most actions unless MFA is present**, but still allows the user to set up MFA and manage their own password.

- IAM → **Policies** → **Create policy** → JSON
- Paste this policy (it’s a common “force MFA” pattern):

```json
{
"Version":"2012-10-17",
"Statement":[
{
"Sid":"AllowViewAccountInfo",
"Effect":"Allow",
"Action":[
"iam:GetAccountPasswordPolicy",
"iam:ListVirtualMFADevices",
"iam:ListUsers",
"iam:ListMFADevices"
],
"Resource":"*"
},
{
"Sid":"AllowManageOwnPasswords",
"Effect":"Allow",
"Action":[
"iam:ChangePassword",
"iam:GetUser"
],
"Resource":"*"
},
{
"Sid":"AllowManageOwnMFA",
"Effect":"Allow",
"Action":[
"iam:CreateVirtualMFADevice",
"iam:DeleteVirtualMFADevice",
"iam:EnableMFADevice",
"iam:DeactivateMFADevice",
"iam:ResyncMFADevice",
"iam:ListMFADevices"
],
"Resource":"*"
},
{
"Sid":"DenyAllExceptListedIfNoMFA",
"Effect":"Deny",
"NotAction":[
"iam:CreateVirtualMFADevice",
"iam:EnableMFADevice",
"iam:GetUser",
"iam:ListMFADevices",
"iam:ListVirtualMFADevices",
"iam:ChangePassword",
"sts:GetSessionToken"
],
"Resource":"*",
"Condition":{
"BoolIfExists":{
"aws:MultiFactorAuthPresent":"false"
}
}
}
]
}

```

- Name the policy: `ForceMFA`
- Now attach it to a group:
    - IAM → **User groups** → Create group `Users`
    - Attach policy `ForceMFA`
    - Add `user-daily` to `Users` **Completed - added mfa to all groups**

✅ Outcome: if someone logs in without MFA, they can’t do anything meaningful. *REVIEW AND TEST*

---

### 6) Create a ReadOnly role + practice switching (10–15 min)

This teaches the **real AWS workflow**: *use a low-privilege user, then assume roles*.

- IAM → **Roles** → Create role
- Trusted entity type: **AWS account**
- Choose: **This account**
- Add permission policy: **ReadOnlyAccess**
- Name role: `ReadOnlyRole`
- Create role **Completed**
- Now allow your daily user to assume it:
    - IAM → Users → `user-daily` → **Add permissions**
    - Create inline policy with:

```json
{
"Version":"2012-10-17",
"Statement":[
{
"Effect":"Allow",
"Action":"sts:AssumeRole",
"Resource":"arn:aws:iam::<YOUR_ACCOUNT_ID>:role/ReadOnlyRole"
}
]
}

```

- Replace `<YOUR_ACCOUNT_ID>` with your account ID.

**Test it:**

- Log in as `user-daily`
- Click your name (top right) → **Switch role**
- Account ID + role name `ReadOnlyRole` → Switch

✅ Outcome: you can safely explore AWS with read-only access.

---

## What you’ll write up for your blog (simple template)

- **Goal:** Set up AWS so I never use root, require MFA, and follow least privilege.
- **What I built:** break-glass admin + daily user + MFA enforcement + ReadOnly role switching
- **What IAM is:** identities, permissions, policies, roles
- **Big lessons:** root is dangerous, MFA reduces risk, roles > permanent permissions

---

## Cost

This project is basically **$0** (IAM is free). Don’t create EC2 or other resources yet.

---

If you want the *even shorter* version (20–30 min), do: **Root MFA + break-glass admin w/MFA + daily user (no admin) + ReadOnly role switch.**

yes or no, this is preparing me to be a cloud security engineer?

**Yes.**

This is exactly the *foundational* IAM work cloud security engineers are expected to understand and apply—especially **least privilege, MFA enforcement, role assumption, and break-glass access**.

for this project answer the following for me:
1.	What I built
2.	Why this matters to security
3.	What could go wrong
4.	How I’d explain this to a non-technical stakeholder

Here are **clean, resume-/blog-ready answers** you can use verbatim or lightly edit.

---

## 1. What I built

I configured a secure AWS Identity and Access Management (IAM) foundation by disabling day-to-day root usage, enforcing multi-factor authentication (MFA), creating a protected “break-glass” administrator account for emergencies, and using a non-admin daily user that assumes limited roles when needed. This setup follows least-privilege principles and mirrors how real production cloud environments manage access.

---

## 2. Why this matters to security

Identity is the primary attack vector in cloud environments. By enforcing MFA, separating emergency admin access from daily use, and requiring role assumption instead of permanent permissions, this design significantly reduces the risk of account takeover, credential misuse, and accidental high-impact changes. It ensures that even if credentials are compromised, attackers are limited in what they can access or modify.

---

## 3. What could go wrong

If MFA is not enforced, a stolen password could lead to full account compromise. If administrators use permanent admin access for daily tasks, mistakes or malicious actions can impact the entire environment. Without a break-glass account, misconfigurations could lock administrators out of their own cloud resources, delaying recovery during an incident.

---

## 4. How I’d explain this to a non-technical stakeholder

Think of it like a building with different keys. The owner’s master key is locked in a safe and only used during emergencies. Employees have regular keys that only open the doors they need, and sensitive areas require a second form of verification, like a badge or fingerprint. This setup reduces the risk of theft, mistakes, and total loss if one key is ever stolen.

---

If you want, next I can:

- Compress this into **LinkedIn-length bullets**
- Turn it into a **portfolio blog post**
- Rewrite it using **cloud-security job language** for resumes
