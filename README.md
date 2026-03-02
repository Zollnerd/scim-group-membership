# SCIM Group Membership Draft

This repo is home to a draft to extend the SCIM 2.0 standard to include a `GroupMember` resource type and a corresponding `/GroupMembers` endpoint. This draft aims to address the scalability issues that face group management in SCIM 2.0 due to the core specification's architecture defining group members as a `members` attribute within the `Group` resource, which does not allow for pagination even when there are an extremely large amount of group membership links. 

## Contributing

I welcome any feedback, either via the IETF SCIM working group's mailing list, direct email (located in the draft), or as an issue created against this repo. 