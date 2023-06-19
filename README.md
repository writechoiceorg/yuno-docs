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

- **Highest**: These are the highest priority issues that severely impact the project or prevent it from progressing. Blocker or critical issues usually indicate showstoppers that need immediate attention and resolution before any other work can proceed.
- **High**: Issues with major or high priority have a significant impact on the project but are not as severe as blockers. They require urgent attention and resolution to keep the project on track.
- **Medium**: Medium priority issues have a moderate impact on the project. They may affect certain functionalities or cause minor inconveniences but do not halt the project's progress entirely. These issues are usually addressed after high-priority tasks.
- **Low**: Low priority issues have minimal impact on the project's progress and are generally non-critical. They are typically minor enhancements, cosmetic changes, or non-essential tasks that can be addressed when higher priority work is complete.
- **Lowest**: Trivial or minor priority issues represent the lowest level of importance. They have minimal impact on the project and are often minor enhancements or cosmetic changes that are nice to have but not crucial to the project's success. These issues are usually handled after all other higher priority tasks.


## Contributing to the GitHub repository
In order to 

