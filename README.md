# Contributing to Yuno Docs

Yudo Docs is now hosted on GitHub for better management and collaboration. Therefore, all the markdown files, images, and OpenAPI specifications will be hosted on and published on Yuno Docs from this repository. This approach allows Yuno and Write Choice to ensure an efficient pipeline of development, as well as better control of what goes up in our documentation. 

In order to ensure an efficient workflow, this documentation will guide you on how to start contributing to Yuno Docs from this repository. 

## Project Board
The first step to start contributing is to be part of the Yuno Docs [project board](https://writechoice.atlassian.net/jira/software/projects/YUNODOCS/boards/2). This project board will be used to control the workflow and assign tasks to the documentation handlers. 

### Columns
- **Backlog**: This column represents the backlog or the issues that are yet to be started.
- **In Progress**: This column in a Kanban board represents the stage of work where team members are actively working on tasks. It is a pivotal column that indicates the status of ongoing work items and helps the team understand what is currently in motion.
- **In Review**: This column represents the issues that are ready to be reviewed by the Write Choice or Yuno team, depending on who was responsible for pushing the issue forward.
- **GitHub Merge / Readme Draft**: When issues reach this stage on the workflow, they're ready to be merged on GitHub and the Readme.io draft is created. **Note**: When issues are moved to this column, Jira will automatically assign Heitor Tessaro to the card since he'll be responsible for merging the branches and making the content available on Readme.io as a draft.
- **Translation**: When the content is already on a draft page/endpoint on Readme.io, it's ready for translation on Localize. **Note**: When issues reach this stage in the workflow, they're automatically assigned to Rafael Moraes, who handles all the translations in Portuguese and Spanish on Localize.  
- **Publishing ✅**: After the content to be developed in the issue has been pushed into Readme.io and also translated on Localize, it is now considered ready to be published and moved to this column. 

### Labels
When creating an issue, please select one or more labels for the card. There are basically three possible labels to be selected:
- **Guides**: Select this label if the issue is related to the User Guides section of the documentation
- **API_Ref**: Select this label if the issue is related to the API Reference section of the documentation
- **Home**: Select this label if the issue is related to the User Home page of Yuno Docs.

### Defining assignees
If you are from the Yuno team and want the guys from Write Choice to work on any issue, let's use the following assigning standard to tick the correct people on the cards:
- **Heitor Tessaro**:
  - Creating and editing pages for the User Guides and API reference
  - Developing HTML code to improve the UX of the documentation
  - Fixing bugs encountered on Readme.io
  - Merging branches and reviewing OpenAPI specification
  - Approving content to be published on Readme.io
- **Rafael Moraes**:
  - Adding and documenting endpoints
  - Creating JSON examples to be added to the endpoints
  - Organizing and translating content on Localize
- **Cassiano Moraes**:
  - Managing the documentation project
  - Developing new documentation-related features
  - Benchmarking and UX improvement

### Choosing priority
In Jira, the priority of an issue card represents its relative importance and urgency within a project. Selecting issue priority is crucial to define what needs to be developed first. Therefore, take a look at the following bullets where each priority is explained:

- **Highest**: These are the highest priority issues that severely impact the project or prevent it from progressing. Highest-priority issues usually indicate showstoppers that need immediate attention and resolution before any other work can proceed.
- **High**: Issues with major or high priority have a significant impact on the project but are not as severe as issues flagged with Highest. They require urgent attention and resolution to keep the project on track.
- **Medium**: Medium priority issues have a moderate impact on the project. They may affect certain functionalities or cause minor inconveniences but do not halt the project's progress entirely. These issues are usually addressed after high-priority tasks.
- **Low**: Low-priority issues have minimal impact on the project's progress and are generally non-critical. They are typically minor enhancements, cosmetic changes, or non-essential tasks that can be addressed when higher-priority work is complete.
- **Lowest**:These issues represent the lowest level of importance. They have minimal impact on the project and are often minor enhancements or cosmetic changes that are nice to have but not crucial to the project's success. These issues are usually handled after all other higher priority tasks.

## Collaborating in the GitHub repository
In order to contribute to this GitHub repository, you have to be added as a collaborator. After being approved, you can start collaborating with Yuno Docs by cloning the repo on your local machine and reading the best practices described in this section. 

### Repository structure
There are three main folders on the Yuno Docs GitHub repository:
- **guides**: In this folder, you can upload or update the content to be published in the Guides section on the public documentation.
- **api_reference**: Here, you can upload or update the OpenAPI specifications that will generate the endpoint's documentation.
- **home**: You can use this folder to upload anything related to the landing page of Yuno Docs. 
- **changelog**: In this folder, you can upload or update the content to be published in the changelog section of the public documentation. Each changelog page will be a .md file.

Inside these main folders, there will be subfolders named with the categories that exist in these main sections, and inside the subfolders, you'll find all the markdown and/or OpenAPI files of that category, as well as a folder named "images", where all the images for that category will be stored.

#### YAML header on Markdown files
In order to sync Markdown files to your Guides, Changelog, or Custom Pages, you'll need to add certain attributes to the top of each page via a YAML front matter block. See below for an example:
```
---
title: Add your title here...
excerpt: Write a small description...
category: ID of the category (for example, GET STARTED WITH YUNO is a category of the Guides section - it has an ID that you can obtain by passing rdme categories --version={project-version} on the CLI)...
---
Write your text here...
```

**Required Attributes**
See below for a table detailing the required YAML front matter attributes:

| Attribute | Required for changelogs? | Required for custompages? | Required for docs? |
|-----------|--------------------------|---------------------------|--------------------|
| title     | Yes                      | Yes                       | Yes                |
| category  | No                       | No                        | Yes                |

### Branches
Aiming to avoid possible issues with the repository, please follow the best practices related to git branches presented below:
- Create a new branch for each task you are working on. This keeps your changes isolated and makes it easier to track progress, review code, and collaborate with other team members.
- Use clear and descriptive names for your branches, indicating the purpose or feature being implemented. Avoid generic names like "feature1" or "branch1" as they provide a little context. For example, if you are reviewing a new endpoint of the Payment API, name your branch like review_create_payment_3ds_endpoint_payment_api.
- Create branches from a stable and up-to-date base branch (usually the main/master branch) to ensure you are working with the latest code. Regularly merge changes from the base branch into your feature branch to keep it up to date.
- Keep each branch focused on a specific feature or task. Smaller, well-defined branches are easier to review, test, and merge. Avoid making unrelated changes within a single branch.
- When your feature or task is complete, open a pull request to initiate a code review. Pull requests provide an opportunity for discussion, feedback, and collaboration before merging your changes into the main branch.
- Once a branch is merged and no longer needed, delete it to declutter the repository. This helps maintain a clean and manageable branch structure.

### Edition process
When performing or suggesting edition to the documentation, you need to follow steps to ensure everyone knows on what you are working on. In addition, these steps ensure that we will ensure the documentation quality adter modifications. The steps are presented below:

#### 1. Clone the repository
Start by cloning the documentation repository to your computer. Open your terminal and use the following command:
```
git clone git@github.com:writechoiceorg/yuno-docs.git
```

To inform others that you are working on editing the documentation, create a card on the Jira board and move it to the **In Progress** column.

#### 2. Create a new branch
Create a new branch where you will make the edits or add new pages. Use the following command:
```
git checkout -b <your-branch-name>
```

#### 3. Edit the documentation
Once you have switched to the new branch, you can start editing the documentation. You have the option to create new files (pages) or make changes to existing ones.

#### 4. Push your contributions to the repository
After completing the edits, send your modifications to the remote repository using the command `git push`. If this is the first time pushing for the current branch, use the following command:
```
git push -u origin <your-branch-name>
```

#### 5. Create a pull request
After pushing your contributions, navigate to the [documentation repository](https://github.com/writechoiceorg/yuno-docs) and create a pull request for your branch. 

Additionally, move the Jira card to the **Review** column to indicate that the pull request is ready for review.

#### 6. Resolve the comments
During the review process, the reviewer may have questions or uncertainties. They will add comments on GitHub to clarify those points. Once the review is complete, you will be notified through the Jira board.

After receiving the notification, address the reviewer's comments promptly. Once addressed, move the Jira card back to the **Review** column.

#### 7. Merge to production
Once the review process is finished, the code owners will handle the remaining steps to merge the modifications into the main branch, making the new version available for Yuno users.

### Protection layers
To protect the documentation from unautorized modification, some protection rules were defined. The following list presents all protections layers defined in GitHub:
- All commits must be made to a non-protected branch and submitted via a pull request. The **main** branch is protected, meaning direct commits to it are not allowed. A pull request requires at least one review and approval from a Code Owner before it can be merged.
- If a pull request has been approved but new commits are pushed afterward, the previous approval is dismissed. To proceed with the updated pull request, a new review and approval from a Code Owner are necessary.
- Reviews must be performed exclusively by Code Owners. Currently, the Write Choice team members are responsible for conducting the reviews.
- Dismissing pull request reviews is restricted and can only be done by the Write Choice team members.
- Lastly, it is required that all conversations related to the code in the pull request are resolved before proceeding with the merge. This ensures that any discussions or issues have been addressed prior to merging the changes.
- Before creating a pull request, it is recommended to ensure that your branch is up to date with the main branch. This can be achieved by executing the following commands in your terminal:
    1. Switch to the main branch:
        ```
        git checkout main
        git pull
        ```
    2. Switch back to your current branch:
        ```
        git checkout <your-branch-name>
        ```
    3. Merge the main branch into your current branch:
        ```
        git merge main
        ```
    4. In case of conflicts between the main branch and your current branch, you will need to resolve them by deciding which content to keep.
    5. After resolving the conflicts, commit the merge:
        ```
        git add .
        git commit -m "Merge master branch into <your-branch-name>"
        ```
    6. Finally, push the changes to the remote repository:

        ```
        git push <your-branch-name>
        ```