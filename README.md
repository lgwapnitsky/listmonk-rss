# Listmonk RSS Newsletter Automation

Automatically send newsletters from RSS feeds using [Listmonk (open
source)](https://listmonk.app) and GitHub Actions (and save $$$ compared to
[Mailchimp](https://mailchimp.com/features/rss-to-email/) or other newsletter
providers.

## Features

- Receive push notifications when newsletters are scheduled 
- Automatically fetch new items from RSS feeds
- Create newsletters based on a Markdown template
- Schedule newsletters with new items with configurable delay
- GitHub Actions integration for automated scheduling

## Requirements

- A running Listmonk instance (e.g., deployed with
  [PikaPods](https://www.pikapods.com))
- GitHub account and a version of this repository
- An existing RSS feed URL, obviously

## Diagram

![Architecture](assets/C4/architecture.png)

## Setup

### 1. Local Setup

1. Fork this repository so that you can set up your own schedule and clone it.

2. Install dependencies using uv:
   ```bash
   uv sync -U
   ```

3. Create a `.env` file with your configuration:
   ```bash
   LISTMONK_API_USER=<your_api_user>
   LISTMONK_API_TOKEN=<your_api_token>

   GH_TOKEN=<your github api token>
   GH_REPOSITORY=<yourusername>/listmonk-rss

   LISTMONK_HOST=<your-listmonk-instance>
   LIST_NAME=<Name of your Listmonk list as it shows in the interface>
   RSS_FEED=<your RSS feed>
   DELAY_SEND_MINS=45
   ```

4. Test the script locally:
   ```bash
   make create_campaign
   ```

### 2. Pushover Notifications (Optional)

To receive notifications when newsletters are scheduled (this gives you an
  opportunity to review the content before it is sent out):

1. Create a Pushover account at https://pushover.net
2. Install the Pushover app on your devices
3. Get your User Key from the Pushover dashboard
4. Create an Application/API Token

Set the env variables:

```bash
PUSHOVER_USER_KEY=<your-pushover-user-key>
PUSHOVER_API_TOKEN=<your-pushover-api-token>
```

### 3. GitHub Actions Setup

1. Create a GitHub Personal Access Token with `repo` scope at
   <https://github.com/settings/tokens>, make sure that you have the following
   permission set: *"Variables" repository permissions (write)* (we need to
   send an [API call to update a repository
   variable](https://docs.github.com/en/rest/actions/variables?apiVersion=2022-11-28#update-a-repository-variable)).

2. Add your `.env` file contents, Pushover credentials, and GitHub token as
   GitHub Secrets in your repository (see screenshots below):
   - Go to Settings → Secrets and variables → Actions
   - Add each environment variable as a new repository secret

2. The workflow is already configured in `.github/workflows/listmonk_rss.yml`
   - Runs on Weekdays at 8:00 UTC
   - Persists state between runs using GitHub repository variables (LAST_UPDATE)
   - Uses GitHub API to store and retrieve the last processed timestamp
   - Automatically creates and schedules newsletters

3. To manually trigger the workflow:
   - Go to Actions → Listmonk RSS
   - Click "Run workflow"

You should run the trigger manually for the first time so that the state can be
saved (make sure you delete the campaign on your Listmonk instance if you don't
want your subscribers to get an email with all existing items). Running the
workflow for the first time will show an error that it wasn't able to find the
artifact - that's expected.

<details>
<summary>
    
### Screenshot "Repository Secrets"
    
</summary>

![](attachments/2025-02-07_18-25-19.png)

</details>

<details>
<summary>
    
#### Screenshot "Repository Variables"

</summary>

![](attachments/2025-02-07_18-26-12.png)

</details>

## Configuration

### Environment Variables

| Variable              | Description                                      | Required |
|-----------------------|--------------------------------------------------|----------|
| LISTMONK_API_USER     | Listmonk API username                            | Yes      |
| LISTMONK_API_TOKEN    | Listmonk API token                               | Yes      |
| LISTMONK_HOST         | Listmonk instance URL                            | Yes      |
| LIST_NAME             | Name of the mailing list in Listmonk             | Yes      |
| RSS_FEED              | URL of the RSS feed to monitor                   | Yes      |
| DELAY_SEND_MINS        | Minutes to delay sending after creation (default: 30) | No       |
| PUSHOVER_USER_KEY     | Pushover user key for notifications (optional)   | No       |
| PUSHOVER_API_TOKEN    | Pushover API token for notifications (optional)  | No       |
| GH_REPOSITORY         | GitHub repository in "owner/repo" format        | Yes      |
| GH_TOKEN              | GitHub token with repo scope for state storage  | Yes      |

### Template Customization

Edit `template.md.j2` to customize your newsletter format. The template uses Jinja2 syntax and has access to:

- `items`: List of RSS feed items with:
  - `title`: Article title
  - `link`: Article URL
  - `summary`: Article summary
  - `media_content`: OpenGraph image URL


## Related work and Contributing

This repo is inspired by
[rss2newsletter](https://github.com/ElliotKillick/rss2newsletter), but I wanted
a solution that runs without a dedicated server (other than GitHub's
infrastructure, of course).

Contributions are welcome, but there's no guarantee that I will be able to act
on them. I use this mostly for my own purposes. My advice would be to fork it
and adjust it to your needs.

## Contact

You may want to subscribe to [my blog](https://blog.heuel.org) 😃.
