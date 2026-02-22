# YAML pipeline editor guide - Azure Pipelines

**Azure DevOps Services | Azure DevOps Server | Azure DevOps Server 2022**

Azure Pipelines provides a YAML pipeline editor that you can use to author and edit your pipelines. The YAML editor is based on the [Monaco Editor](https://github.com/microsoft/monaco-editor). The editor provides tools like Intellisense support and a task assistant to provide guidance while you edit a pipeline.

This article shows you how to edit your pipelines using the YAML Pipeline editor, but you can also edit pipelines by modifying the **azure-pipelines.yml** file directly in your pipeline's repository using a text editor of your choice, or by using a tool like Visual Studio Code and the [Azure Pipelines for VS Code](https://github.com/Microsoft/azure-pipelines-vscode) extension.

## Edit a YAML pipeline

To access the YAML pipeline editor, do the following steps.

1.  Sign in to your organization (`https://dev.azure.com/{yourorganization}`).
    
2.  Select your project, choose **Pipelines**, and then select the pipeline you want to edit. You can browse pipelines by **Recent**, **All**, and **Runs**. For more information, see [view and manage your pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/create-first-pipeline?view=azure-devops#view-and-manage-your-pipelines).
    
    ![Azure Pipelines landing page.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-landing-page.png?view=azure-devops)
    
3.  Choose **Edit**.
    
    ![Azure Pipelines YAML edit button.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-edit.png?view=azure-devops)
    
    Important
    
    The YAML pipeline editor is only available for YAML pipelines. If you are presented with a graphical user interface when you choose **Edit**, your pipeline was created using the classic pipeline designer. For information on converting your classic pipelines to YAML, see [Migrate your Classic pipeline to YAML](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/from-classic-pipelines?view=azure-devops).
    
4.  Make edits to your pipeline using [Intellisense](#use-keyboard-shortcuts) and the [task assistant](#use-task-assistant) for guidance.
    
    ![YAML pipeline editor.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-editor.png?view=azure-devops)
    

5.  Choose **Save**. You can commit directly to your branch, or create a new branch and optionally start a pull request.
    
    ![YAML pipeline editor save window.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-save.png?view=azure-devops)
    

5.  Choose **Validate and save**. You can commit directly to your branch, or create a new branch and optionally start a pull request.
    
    ![Screenshot showing the YAML pipeline editor validate and save window.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-validate-and-save.png?view=azure-devops)
    

### Use keyboard shortcuts

The YAML pipeline editor provides several keyboard shortcuts, which we show in the following examples.

-   Choose **Ctrl**+**Space** for Intellisense support while you're editing the YAML pipeline.
    
    ![YAML pipeline editor intellisense.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-intellisense.png?view=azure-devops)
    
-   Choose **F1** (**Fn+F1** on Mac) to display the command palette and view the available keyboard shortcuts.
    
    ![YAML pipeline editor command palette.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-editor-command-palette.png?view=azure-devops)
    

## Use task assistant

The task assistant provides a method for adding tasks to your YAML pipeline.

-   To display the task assistant, edit your YAML pipeline and choose **Show assistant**.
    
    ![Show ask assistant for editing YAML pipelines.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-task-assistant-show.png?view=azure-devops)
    
-   To hide the task assistant, choose **Hide assistant**.
    
    ![Hide task assistant for editing YAML pipelines.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-task-assistant-hide.png?view=azure-devops)
    
-   To use the task assistant, browse or search for tasks in the **Tasks** pane.
    
    ![Task assistant search.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-task-assistant-search.png?view=azure-devops)
    
-   Select the desired task and configure its inputs.
    
    ![Task assistant add.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-task-assistant-add.png?view=azure-devops)
    
-   Choose **Add** to insert the task YAML into your pipeline.
    

![Task assistant added.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-task-assistant-task-added.png?view=azure-devops)

-   You can edit the YAML to make more configuration changes to the task, or you can choose **Settings** above the task in the YAML pipeline editor to configure the inserted task in the task assistant.

## Validate

Validate your changes to catch syntax errors in your pipeline that prevent it from starting. Choose **More actions** > **Validate**.

![Validate and Download full YAML.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-validate.png?view=azure-devops)

Azure Pipelines validates your pipelines each time you save. Choose **Validate and save** to validate your pipeline before saving. If there are any errors, you can **Cancel** or **Save anyway**. To save your pipeline without validating, choose **Save without validating**.

![Screenshot showing the Validate and save button.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-validate-and-save-button.png?view=azure-devops)

Azure Pipelines detects incorrect variable definitions defined at the pipeline, stage, and job level and detects incorrect YAML conditions defined at the pipeline, stage, and job level.

## Download full YAML

You can [preview the fully parsed YAML document](/en-us/azure/devops/release-notes/2020/sprint-165-update#preview-fully-parsed-yaml-document-without-committing-or-running-the-pipeline) without committing or running the pipeline. Choose **More actions** > **Download full YAML**.

![Validate and Download full YAML.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-validate.png?view=azure-devops)

**Download full YAML** [Runs](/en-us/rest/api/azure/devops/pipelines/runs/run%20pipeline?view=azure-devops-rest-6.1&preserve-view=true) the Azure DevOps REST API for Azure Pipelines and initiates a download of the rendered YAML from the editor.

## Manage pipeline variables

You can manage pipeline variables both from within your YAML pipeline and from the pipeline settings UI.

To manage pipeline variables, do the following steps.

1.  Edit your YAML pipeline and choose **Variables** to manage pipeline variables.
    
    ![Manage pipeline variables button.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-variables-button.png?view=azure-devops)
    
2.  Choose from the following functions:
    
    ![Manage pipeline variables in the YAML editor.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-editor-manage-variables.png?view=azure-devops)
    

To manage pipelines variables in the pipeline settings UI, do the following steps.

1.  Edit the pipeline and choose **More actions** > **Triggers**.
    
    ![Pipeline settings UI menu.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-settings-ui-menu.png?view=azure-devops)
    
2.  Choose **Variables**.
    
    ![Pipeline settings UI for variables.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-settings-ui-variables.png?view=azure-devops)
    

For more information on working with pipeline variables, see [Define variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops).

## Configure the default agent pool

If a YAML pipeline doesn't specify an agent pool, the agent pool configured in the **Default agent pool for YAML** setting is used. This pool is also used for post-run cleanup tasks.

To view and configure the **Default agent pool for YAML** setting:

1.  Edit the pipeline and choose **More actions** > **Triggers**.
    
    ![Screenshot of the pipeline settings UI menu.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-settings-ui-menu.png?view=azure-devops)
    
2.  Choose **YAML**, and select the desired agent pool using the **Default agent pool for YAML** dropdown list.
    
    ![Screenshot of the default agent pool for YAML pipelines.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/default-agent-pool-for-yaml.png?view=azure-devops)
    

**Default agent pool for YAML** is configured on a per-pipeline basis.

## Manage settings using the pipeline settings UI

Some YAML pipeline settings are configured using the pipeline settings UI instead of in the YAML file.

1.  Edit the pipeline and choose **More actions** > **Triggers**.
    
    ![Screenshot of the pipeline settings UI menu.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-settings-ui-menu.png?view=azure-devops)
    
2.  From the pipeline settings UI, choose the tab of the setting to configure.
    
    ![Screenshot of the pipeline settings UI for triggers.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/pipeline-settings-ui-triggers.png?view=azure-devops)
    

## View and edit templates

Note

This feature is available starting in Azure DevOps Server 2022.1.

[Templates](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops) are a commonly used feature in YAML pipelines. They're an easy way to share pipeline snippets and are a powerful mechanism for verifying and enforcing [security and governance](https://learn.microsoft.com/en-us/azure/devops/pipelines/security/templates?view=azure-devops) in your pipeline. Previously, the editor didn't support templates, so authors of YAML pipelines couldn't get intellisense assistance. Now Azure Pipelines supports a YAML editor, for which we're previewing support. To enable this preview, [go to preview features](https://learn.microsoft.com/en-us/azure/devops/pipelines/https://learn.microsoft.com/en-us/azure/devops/pipelines/project/navigation/preview-features?view=azure-devops) in your Azure DevOps organization, and enable **YAML templates editor**.

Important

This feature has the following limitations.

-   If the template has required parameters that aren't provided as inputs in the main YAML file, then the validation fails and prompts you to provide those inputs.
    
-   You can't create a new template from the editor. You can only use or edit existing templates.
    

As you edit your main Azure Pipelines YAML file, you can either _include_ or _extend_ a template. As you enter the name of your template, you may be prompted to validate your template. Once validated, the YAML editor understands the schema of the template, including the input parameters.

![YAML template.](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/yaml-pipeline-editor/yaml-pipeline-editor-templates.png?view=azure-devops)

Post validation, you can go into the template by choosing **View template**, which opens the template in a new browser tab. You can make changes to the template using all the features of the YAML editor.
