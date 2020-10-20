---
layout: page
title: Fapi-Policies
permalink: /fapipolicies/
---
This is an editor for FAPI policies. They can be imported using `Fapi_Import()` or the
`tss2_import` command line utility.

WARNING: This is still work in progress and subject to change.

<div id='editor_holder'></div>

## JSON output

<textarea id='output' style="width: 100%; height: 30em; font-family: monospace;"></textarea>
<button id='update'>Update Editor</button>

<link rel='stylesheet' href='https://unpkg.com/spectre.css/dist/spectre.min.css'>
<link rel='stylesheet' href='https://unpkg.com/spectre.css/dist/spectre-icons.min.css'>

<script src="https://cdn.jsdelivr.net/npm/@json-editor/json-editor@latest/dist/jsoneditor.min.js"></script>

<script>
var options = {
    ajax: true,
    disable_collapse: true,
    no_additional_properties: true,
    show_opt_in: true,
    disable_edit_json: true,
    disable_properties: true,
    theme: "spectre",
    iconlib: "spectre",
};

fetch("../tpm2-tss-policy-editor.schema.json")
    .then(function(response) {
        if (response.ok) {
            return response.json();
        } else {
            console.log(response);
            throw new Error("Bad response");
        }
    })
    .then(function(json) {
        options.schema = json;
        var editor = new JSONEditor(document.getElementById('editor_holder'), options);
        editor.on('change', function () {
              let json = editor.getValue();
              document.getElementById('output').value = JSON.stringify(json, null, 2);
            });
        document.getElementById('update').addEventListener('click', function () {
                editor.setValue(JSON.parse(document.getElementById('output').value));
            });

    });
</script>
