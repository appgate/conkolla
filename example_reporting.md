# Use conkolla to export or report on data
You can use conkolla to export the responses from any API call. You can either downlod it as:
- JSON
- CSV

CSV data will be flattened by one level only.

You can for example use it to report on licenses used, active sessions or any other information of interest. Other information, such as the "stats" page can also be fetched as CSV data.

## Howto:
1. Download [Conkolla latest release](https://github.com/appgate/conkolla/releases/latest) 
1. Launch it (Windows in this example): `Conkolla.exe --authName <a username> --authPassword <a password>`
1. Open chrome browser and go to [https://localhost:4433](https://localhost:4433)
1. Log into your controller by clicking on the **+** next to _connected controllers_. Assure you login with an admin user.
1. When succesfully logged in, click on "Start making rest calls by clicking here".
1. In the form type the API path you want to make a call for, example `/stats/active-sessions`, and click on GET.
1. The result is presented in the below ace editor.
1. Save the content as CSV by clicking the button `save csv`

for more options and information of usage see the [documentation](./README.md).
