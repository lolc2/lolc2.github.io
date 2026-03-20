##### Abusing Microsoft SharePoint for C2

Attackers can exploit SharePoint's lists and document libraries to store commands and exchange data, blending C2 traffic with normal enterprise document management.

![sharepoint](/doc/sharepoint.png)

#### How It Works

SharePoint-based C2 uses the **Microsoft Graph API** to interact with SharePoint sites, lists, and list items. Commands are stored in list items; the agent polls for new items, executes commands, and updates items with results.

**GraphStrike** integration: The GraphStrike project enables Cobalt Strike communication over Microsoft Graph, including SharePoint lists as a C2 channel. File names containing the identifier `pD9-tK` in O365 logs may indicate GraphStrike activity.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser/non-Office processes making Graph API calls to `/sites/*/lists/`
2. **List item anomalies:** SharePoint list items containing encoded or structured command/response data
3. **GraphStrike indicator:** File names containing the string `pD9-tK` in O365 audit logs
