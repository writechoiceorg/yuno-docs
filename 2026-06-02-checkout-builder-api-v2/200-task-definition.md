# 🚀 Yuno Docs Agent Task Definition

## 🗄️ Project Context
- **Migration**: We recently migrated https://docs.y.uno/ from Readme to Mintlify.
- **Site URL**: https://docs.y.uno/
- **Source**: Production files are in `/005-yuno-mintlify-docs`.
- **Legacy Reference**: Use `/100-archive/legacy-docs/004-yuno-readme-docs-legacy` ONLY for historical data.

## 🎯 Task Workflow
1. **Creation**: Create a task folder in `001-in-progress-tasks` using the format `YYYY-MM-DD-short-task-name` (no spaces).
2. **Template**: Base the folder structure on `XXXX-XX-XX-template-task-folder`.
3. **Context Compression (CRITICAL)**:
    - Create a `001-context` directory.
    - Move the initial raw request/Slack thread to `001-context/.context-raw/100-request.md`.
    - Create `001-context/100-request-summary.md` with only actionable technical requirements.
4. **Implementation**: Always work directly in the root `/005-yuno-mintlify-docs` to ensure the primary source is always updated and ready for PRs.
5. **Backups (MANDATORY)**: As a final step before completion, copy all modified files into the corresponding `/005-yuno-mintlify-docs` subfolder within your task directory. This serves as a local record and mandatory backup of your specific contribution.

## 🛠️ Implementation Standards
- **OpenAPI Optimization**: Never read large JSON samples (e.g., `yuno_sandbox_tested.json`) in full. Use `grep` or `jq` to extract specific endpoints.
- **Consistency**: Analyze if the request is technically consistent and ask clarifying questions before starting.
- **Definition of Done**: 
    - Prepare a concise PR description in `pr.md`.
    - Mark all items in your `TASKS.md` checklist.
    - **Stakeholder Communication**: Save any draft messages for Slack/email into a `slack-messages.md` file within the task folder.
    - When finished, prepare a summary response for the user.

## 📦 Maintenance
- **Archive Policy**: Tasks older than 7 days will be moved to `100-archive/finished-tasks/` to keep the primary directories clean.

---


Thread

[yuno-docs](https://writechoice.slack.com/archives/C03AF1FRNQZ/p1779128979025619)

Aleksandra Todorovic [May 18th at 3:29 PM](https://writechoice.slack.com/archives/C03AF1FRNQZ/p1779128979025619)  

Management API Checkout Builder  ![:thread:](https://a.slack-edge.com/production-standard-emoji-assets/16.0/google-medium/1f9f5@2x.png)

8 replies

----------

Aleksandra Todorovic [May 18th at 3:32 PM](https://writechoice.slack.com/archives/C03AF1FRNQZ/p1779129157214239?thread_ts=1779128979.025619&cid=C03AF1FRNQZ)  

hi  [@Heitor](https://writechoice.slack.com/team/U02QWCKQNEB)  [@Damián](https://writechoice.slack.com/team/U07FB71NRDL)  
could you please add this to a  **hidden**  page in the API reference as a new dropdown option called "Checkout Builder"?let me know if you need anything else![https://docs.y.uno/reference/payment-methods-checkout/the-payment-method-object-checkout](https://docs.y.uno/reference/payment-methods-checkout/the-payment-method-object-checkout)cc  [@Romel](https://writechoice.slack.com/team/U037S9U4F7B)  [@Daniel Vega](https://writechoice.slack.com/team/U09QBGLHA3C)  [@Marco Santarelli](https://writechoice.slack.com/team/U0AH0L9PBBJ)

Yuno_Checkout_Builder_Management_API_Reference.md

#  Yuno  Checkout  Builder  —  Management  API  Reference

​

>  **Scope:**  Programmatic  configuration  of  Yuno  checkouts  via  the  Dashboard  BFF  —  the  same  API  that  powers  the  Checkout  Builder  UI  in  the  Yuno  dashboard.

>

>  **Status:**  No  new  endpoints  required  —  everything  described  here  already  exists  in  production.

Click to expand inline (1,061 lines)

Yuno API Documentation

[The Payment Method Object - Yuno API Documentation](https://docs.y.uno/reference/payment-methods-checkout/the-payment-method-object-checkout)

Damián [May 18th at 3:33 PM](https://writechoice.slack.com/archives/C03AF1FRNQZ/p1779129220202179?thread_ts=1779128979.025619&cid=C03AF1FRNQZ)  

Hi  [@Aleksandra Todorovic](https://writechoice.slack.com/team/U09KDNDLVT7), we will work on it.  
cc:  [@Douglas](https://writechoice.slack.com/team/U0ADPFQBUH2)

Douglas [Wednesday at 10:25 AM](https://writechoice.slack.com/archives/C03AF1FRNQZ/p1779888338384419?thread_ts=1779128979.025619&cid=C03AF1FRNQZ)  

Hi  [@Aleksandra Todorovic](https://writechoice.slack.com/team/U09KDNDLVT7), sorry for the delay! Just wanted to let you know that the Checkout Builder API reference pages have been merged and are now live. Since they're hidden from the sidebar, you can access them directly through these links: - Overview:  [https://docs.y.uno/reference/checkout-builder/overview](https://docs.y.uno/reference/checkout-builder/overview)  
- Fetch Checkout Configuration:  [https://docs.y.uno/reference/checkout-builder/fetch-checkout-configuration](https://docs.y.uno/reference/checkout-builder/fetch-checkout-configuration)  
- Publish Checkout Configuration:  [https://docs.y.uno/reference/checkout-builder/publish-checkout-configuration](https://docs.y.uno/reference/checkout-builder/publish-checkout-configuration)  
- Get Required Fields:  [https://docs.y.uno/reference/checkout-builder/get-required-fields](https://docs.y.uno/reference/checkout-builder/get-required-fields)  
- Get Country Data:  [https://docs.y.uno/reference/checkout-builder/get-country-data](https://docs.y.uno/reference/checkout-builder/get-country-data)  
- Fetch Styling & SDK Settings:  [https://docs.y.uno/reference/checkout-builder/fetch-styling-settings](https://docs.y.uno/reference/checkout-builder/fetch-styling-settings)  
- Update Styling & SDK Settings:  [https://docs.y.uno/reference/checkout-builder/update-styling-settings](https://docs.y.uno/reference/checkout-builder/update-styling-settings)Let us know if you have any feedback!  ![:raised_hands:](https://a.slack-edge.com/production-standard-emoji-assets/16.0/google-medium/1f64c@2x.png) (edited)

[10:26](https://writechoice.slack.com/archives/C03AF1FRNQZ/p1779888370332139?thread_ts=1779128979.025619&cid=C03AF1FRNQZ)

cc:  [@Damián](https://writechoice.slack.com/team/U07FB71NRDL)  [@Heitor](https://writechoice.slack.com/team/U02QWCKQNEB)

Aleksandra Todorovic [Thursday at 9:50 AM](https://writechoice.slack.com/archives/C03AF1FRNQZ/p1779972636802949?thread_ts=1779128979.025619&cid=C03AF1FRNQZ)  

hi  [@Douglas](https://writechoice.slack.com/team/U0ADPFQBUH2)  sorry, forgot to get back to you, we will need to adapt this soon, we got some changes. will let you know in a bit

Douglas [Thursday at 10:09 AM](https://writechoice.slack.com/archives/C03AF1FRNQZ/p1779973751817609?thread_ts=1779128979.025619&cid=C03AF1FRNQZ)  

Hi  [@Aleksandra Todorovic](https://writechoice.slack.com/team/U09KDNDLVT7), no worries at all! Once you send over the information, we’ll update it  ![:raised_hands:](https://a.slack-edge.com/production-standard-emoji-assets/16.0/google-medium/1f64c@2x.png)

Aleksandra Todorovic [Today at 11:21 AM](https://writechoice.slack.com/archives/C03AF1FRNQZ/p1780410060949919?thread_ts=1779128979.025619&cid=C03AF1FRNQZ)  

hi  [@Douglas](https://writechoice.slack.com/team/U0ADPFQBUH2)  we finally have it, here you go: can you rework what you did above and let me know once its published to the hidden page?thanks!cc  [@Daniel Vega](https://writechoice.slack.com/team/U09QBGLHA3C)

Yuno_Checkout_Builder_Management_API_Reference (1).md

#  Yuno  Checkout  Builder  —  Management  API  Reference

​

>  **Scope:**  Programmatic  configuration  of  Yuno  checkouts  via  Yuno's  public  B2B  API.

>

>  **Status:**  BETA  —  endpoints  are  stable  in  shape  but  subject  to  additive  changes  during  the  BETA  window.  Open  only  to  allowlisted  merchants.  Contact  your  Yuno  TAM  for  access.

Click to expand inline (1,057 lines)

Saved for later

Damián [Today at 11:22 AM](https://writechoice.slack.com/archives/C03AF1FRNQZ/p1780410158980669?thread_ts=1779128979.025619&cid=C03AF1FRNQZ)  

Hi  [@Aleksandra Todorovic](https://writechoice.slack.com/team/U09KDNDLVT7), Douglas is OOO today, but I'll work on it.