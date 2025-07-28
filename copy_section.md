# 🚀  CI/CD Pipeline for IBM BAW Project Installation

This workflow automates the process of installing an IBM Business Automation Workflow (BAW) project when a new `.json` configuration file is pushed to the `main` branch inside the `workflow/CO` folder.

---

## 📁  Trigger Conditions

The workflow is triggered when:

- A file matching `workflow/CO/**.json` is pushed to the `main` branch.
- Or manually via the `workflow_dispatch` button on GitHub.

---

## 🧪  Workflow Steps

### 🧩 1. Checkout Repository

```yaml
- name: Checkout code
  uses: actions/checkout@v4
```

### 🧾 2. Extract Parameters from JSON

```yaml
- name: Extract values from json file
  run: |
    newest_json=$(ls -t workflow/CO/*.json | head -n 1)
    server_url=$(jq -r '.server_url' $newest_json)
    snapshot_acronym=$(jq -r '.snapshot_acronym' $newest_json)
    project_short_name=$( jq -r '.project_short_name' $newest_json)
    snapshot_description=$( jq -r '.snapshot_description' $newest_json)
    snapshot_name=$( jq -r '.snapshot_name' $newest_json)
    project_name=$( jq -r '.project_name' $newest_json)
    echo "server_url=$server_url" >> $GITHUB_OUTPUT
    echo "snapshot_acronym=$snapshot_acronym" >> $GITHUB_OUTPUT
    echo "project_short_name=$project_short_name" >> $GITHUB_OUTPUT
    echo "snapshot_description=$snapshot_description" >> $GITHUB_OUTPUT
    echo "snapshot_name=$snapshot_name" >> $GITHUB_OUTPUT
    echo "project_name=$project_name" >> $GITHUB_OUTPUT
```

### 🔐 3. Get CSRF Token for Authenticated API Calls

```yaml
- name: Get CSRF token
  run: |
    echo "Project Name: ${{ steps.extract_json.outputs.project_name }}"
    response=$(curl -s -X POST ${{ steps.extract_json.outputs.server_url }}/bas/ops/system/login \
    -H "Authorization: ZenApiKey ${{ secrets.MY_SECRET }}" \
    -H "Content-Type: application/json" \
    -d '{ "refresh_groups": true, "requested_lifetime": 7200}')
    echo "Response: $response"
    csrfkey=$(echo "$response" | jq -r '.csrf_token')
    echo "Extracted csrfkey: $csrfkey"
    echo "csrfkey=$csrfkey" >> $GITHUB_OUTPUT
```


### 📦 4. Create the Offline Installation Package

```yaml
- name: Create installation package
  run: |
    echo "Project Name: ${{ steps.extract_json.outputs.project_name }}"
    echo "Create Package for the following profile : ${{ secrets.target_profile }}"
    response=$(curl -s -X POST ${{ steps.extract_json.outputs.server_url }}/bas/ops/std/bpm/containers/${{ steps.extract_json.outputs.project_short_name }}/versions/${{ steps.extract_json.outputs.snapshot_acronym }}/offline_package?server=${{ secrets.target_profile }} \
    -H "Authorization: ZenApiKey ${{ secrets.MY_SECRET }}" \
    -H "Content-Type: application/json" \
    -H "BPMCSRFToken: ${{ steps.csrf_token.outputs.csrfkey }}" )
    echo "Response: $response"
    url=$(echo "$response" | jq -r '.url')
    echo "offline package url: $url"
    echo "joburl=$url" >> $GITHUB_OUTPUT

```

### ⏱ 5. Monitor Package Creation Job Status

```yaml
- name: Check Status of the package creation
  run: |
    echo "Job URL  : ${{ steps.create_package.outputs.joburl }}"
    end=$((SECONDS+240))
    while true; do
    response=$(curl -s -X GET ${{ steps.create_package.outputs.joburl }} \
    -H "Authorization: ZenApiKey ${{ secrets.MY_SECRET }}" \
    -H "Content-Type: application/json" \
    -H "BPMCSRFToken: ${{ steps.csrf_token.outputs.csrfkey }}" )
    echo "Response: $response"
    state=$(echo "$response" | jq -r '.state')
    echo "The status of the package creation is: $state"
    if [[ "$state" == "success"  ]]; then
        echo "Package creation successfully."
        break
        fi
    if [[ "$state" == "failure" ]]; then
    echo "Package creation failed."
    exit 1
    fi
    if (( SECONDS >= end )); then
    echo "Timeout reached (4 minutes). Exiting loop."
    exit 1
    break
    fi
    sleep 10
    done
```


### ⬇️ 6. Download the Installation Package

```yaml
- name: Download the package
  run: |
    echo "Job URL  : ${{ steps.extract_json.outputs.server_url }}/bas/ops/std/bpm/containers/${{ steps.extract_json.outputs.project_short_name }}/versions/${{ steps.extract_json.outputs.snapshot_acronym }}/install_package?server=Production"      
    curl -X GET ${{ steps.extract_json.outputs.server_url }}/bas/ops/std/bpm/containers/${{ steps.extract_json.outputs.project_short_name }}/versions/${{ steps.extract_json.outputs.snapshot_acronym }}/install_package?server=Production \
    -o workflow/CO/${{ steps.extract_json.outputs.snapshot_acronym }}_Installpck.zip \
    -H "Authorization: ZenApiKey ${{ secrets.MY_SECRET }}" \
    -H "BPMCSRFToken: ${{ steps.csrf_token.outputs.csrfkey }}" \
    -H "Accept: application/octet-stream"   
    echo "Package downloaded successfully."   

```

### ⚙️ 7. Install the Package

```yaml
- name: Install the package
  run: |
    echo "Install the package in the server"
    echo "Project Name: ${{ steps.extract_json.outputs.project_name }}"
    response=$(curl -X POST "${{ steps.extract_json.outputs.server_url }}/bas/ops/std/bpm/containers/install?inactive=false&caseOverwrite=false" -F 'install_file=@workflow/CO/${{ steps.extract_json.outputs.snapshot_acronym }}_Installpck.zip' \
    -H "Authorization: ZenApiKey ${{ secrets.MY_SECRET }}" \
    -H "Accept: application/json" \
    -H "BPMCSRFToken: ${{ steps.csrf_token.outputs.csrfkey }}" )
    echo "Response: $response"
    url=$(echo "$response" | jq -r '.url')
    echo "install queue package url: $url"
    echo "joburl=$url" >> $GITHUB_OUTPUT
```

### 🔄 8. Monitor Installation Job Status

```yaml
- name: Check Status of the package installation
  run: |
    echo "Job URL  : ${{ steps.install_package.outputs.joburl }}"
    end=$((SECONDS+240))
    while true; do
    response=$(curl -s -X GET ${{ steps.install_package.outputs.joburl }} \
    -H "Authorization: ZenApiKey ${{ secrets.MY_SECRET }}" \
    -H "Content-Type: application/json" \
    -H "BPMCSRFToken: ${{ steps.csrf_token.outputs.csrfkey }}" )
    echo "Response: $response"
    state=$(echo "$response" | jq -r '.state')
    echo "The status of the package creation is: $state"
    if [[ "$state" == "success"  ]]; then
    break
    fi
    if [[ "$state" == "failure" ]]; then
        echo "Package installation failed."
        exit 1
    fi
    if (( SECONDS >= end )); then
    echo "Timeout reached (4 minutes). Exiting loop."
        exit 1
    break
    fi
    sleep 10
    done
```