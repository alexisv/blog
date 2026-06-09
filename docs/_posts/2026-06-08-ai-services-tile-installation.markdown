---
layout: post
title:  "AI Services Installation"
date:   2026-06-08 05:59:00 -0400
categories: deephackmode.io update
---
Now that I have an Elastic Application Runtime foundation available and ready, I want to try the AI Services tile.  As per the official documentation, AI Services will enable the use of LLMs (Large Language Models) in the applications.  In case you've been hiding under a rock in the past couple of years, LLM is a type of artificial intelligence trained on enormous amounts of text (books, websites, articles, conversatons, code and more) to understand and generate human-like language.  Let's get into it!

To get started, I downloaded two artifacts from the Broadcom Download page:

1. VMware Tanzu for Postgres on Tanzu Platform 10.4.1 (postgres-10.4.1.pivotal)
2. AI Services for VMware Tanzu Platform 10.4.1 (genai-10.4.1.pivotal)

Postgres tile installation

In the Foundation Core, I navigated to Manage->Capabilities and clicked "Import Capability".  I selected the "postgres-10.4.1.pivotal" file, and waited until it was showing in the "Available" section.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/postgres-available.png" alt="Import Postgres" title="Import Postgres">
</div>
<figcaption>Imported the Postgres tile</figcaption>
</figure> 

Clicked on the context menu, and then clicked "Stage".
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/postgres-staged.png" alt="Stage Postgres" title="Stage Postgres">
</div>
<figcaption>Staged the Postgres tile</figcaption>
</figure> 

Clicked on the tile and that took me to the settings page.

In the "Assign AZs and Networks" tab, I configured the jobs to use the "az1" AZ.  The Network and Service Network were both set to "deployment-network".  I only have one AZ and one network, so there's not much to decide here.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/postgres-assign-azs-and-networks.png" alt="Assign AZs and Networks" title="Assign AZs and Networks">
</div>
<figcaption>Assign AZs and Networks</figcaption>
</figure>

In the "On-Demand Plans" tab, I added a plan named "on-demand-postgres-db".  I set the "AZs to deploy..." to "az1".  Made sure that the "Server VM type" is "large".  All other settings in the plan were left as is.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/postgres-on-demand-plans-2.png" alt="On-demand plan name" title="On-demand plan name">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/postgres-on-demand-plans-3.png" alt="On-demand plan AZ" title="On-demand plan AZ">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/postgres-on-demand-plans-4.png" alt="On-demand plan VM type" title="On-demand plan VM type">
</div>
<figcaption>Added an on-demand plan</figcaption>
</figure>

In the "Dynamic Plan Settings" tab, I set the "AZs to deploy postgres ..." to "az1".
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/postgres-dynamic-plan-settings.png" alt="Dynamic Plan Settings" title="Dynamic Plan Settings">
</div>
<figcaption>Dynamic Plan Settings: AZs to deploy postgres</figcaption>
</figure>

Navigated to Manage->Capabilities->Review Pending Changes.  Then, click "Apply Pending Changes".  Monitored the AC (Apply Changes) until it completed successfully!
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/foundation-core-review-pending-changes.png" alt="Review Pending Changes" title="Review Pending Changes">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/foundation-core-apply-pending-changes.png" alt="Apply Pending Changes" title="Apply Pending Changes">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/foundation-core-apply-changes-in-progress.png" alt="AC in progress" title="AC in progress">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/foundation-core-apply-changes-in-completed.png" alt="AC completed!" title="AC completed!">
</div>
<figcaption>Apply Changes!</figcaption>
</figure>

Now let's install the AI Services tile.

Same deal as before when installing a tile!  Clicked "Import Capability".  Selected the AI Services tile file.  Waited for the Importing to complete.  Staged the tile.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/foundation-core-import-button.png" alt="Import Capability" title="Import Capability">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/foundation-core-importing.png" alt="Importing" title="Importing">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-imported.png" alt="AI Services available" title="AI Services available">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-staged.png" alt="AI Services staged" title="AI Services staged">
</div>
<figcaption>Imported and Staged the AI Services tile</figcaption>
</figure>

Clicked the AI Services tile, to start the configuration.

In the "Assign AZs and Networks", configured the jobs to use "az1" as the AZ, and the "deployment-network" as the "Network" and "Service Network".
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-assign-azs-and-networks.png" alt="Assign AZs and Networks" title="Assign AZs and Networks">
</div>
<figcaption>AI Services tile: Assign AZs and Networks</figcaption>
</figure>


In the "On Platform Models" tab, I added an Ollama Model, [gemma4](https://ollama.com/library/gemma4){:target="_blank"}.  Set Model name as "gemma4:e4b".  The Handle was set as "gemma4:e4b".  The Model Capabilities were set to "Chat, Tools".  Set "VM Type" to "cpu".  Set AZ to "az1".  Set Disk Size to 50GB.  All the other settings were left as they were.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-on-platform-models-add-gemma4.png" alt="Add Ollama Model" title="Add Ollama Model">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-on-platform-models-add-gemma4-1.png" alt="Add Ollama Model part 2" title="Add Ollama Model part 2">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-on-platform-models-add-gemma4-1-b.png" alt="Add Ollama Model part 3" title="Add Ollama Model part 3">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-on-platform-models-add-gemma4-2.png" alt="Add Ollama Model part 4" title="Add Ollama Model part 4">
</div>
<figcaption>AI Services tile: Add Ollama Model</figcaption>
</figure>

Next, let's configure a plan.  In the "Plan Config", I added a plan named "gemma4-plan".  I set Models to "gemma4:e4b".  Left other settings as they were.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-plan-config-add-gemma4-plan.png" alt="Add gemma4-plan" title="Add gemma4-plan">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-plan-config-add-gemma4-plan-2.png" alt="Add gemma4-plan part 2" title="Add gemma4-plan part 2">
</div>
<figcaption>AI Services tile: Add a Plan</figcaption>
</figure>

In the "Advanced Config" tab, I unchecked the "Strict mode for AI Server" checkbox.  That's the only thing I changed there.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-advanced-config-uncheck-strict-mode.png" alt="Advanced Config" title="Advanced Config">
</div>
<figcaption>Disable Strict mode for AI Server</figcaption>
</figure>

In the "Database Config" tab, for the "AI Server Database Source", I kept it as "Service Broker", set the Offering Name to "postgres", and the Plan Name to "on-demand-postgres-db".  I did the same thing for the "MCP Gateway Database Source".
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-database-config.png" alt="AI Server Database settings" title="AI Server Database settings">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-database-config-mcp-gateway.png" alt="MCP Gateway Database settings" title="MCP Gateway Database settings">
</div>
<figcaption>AI Services tile: Database Config</figcaption>
</figure>


In the "Errands" tab, I only switched the setting for "Install Agent Buildpack" to on.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-errands-install-agent-buildpack.png" alt="Install Agent Buildpack errand enabled" title="Install Agent Buildpack errand enabled">
</div>
<figcaption>Install Agent Buildpack errand enabled</figcaption>
</figure>

The other settings in the other tabs were left as they were (default settings).  At this point, the AI Services tile has been fully configured.  Apply Changes has been started.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-configured.png" alt="AI Services tile configured" title="AI Services tile configured">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-apply-pending-changes.png" alt="Apply Pending Changes" title="Apply Pending Changes">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-apply-changes-in-progress.png" alt="AC in progress" title="AC in progress">
</div>
<figcaption>AI Services tile: Applying Changes</figcaption>
</figure>

After a little more than 45 minutes, the AC completed successfully.  All errands were successful so that's a great thing.
<figure>
<div class="image-row-big">
<img class="popup-img" src="/assets/images/2026-06-08-ai-services-installation/ai-services-apply-changes-completed.png" alt="AC completed" title="AC completed">
</div>
<figcaption>AI Services tile: AC completed</figcaption>
</figure>

