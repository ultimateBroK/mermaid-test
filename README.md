graph TD
    A[Bắt đầu - Mở trang Users Module] --> B{Check Authentication};
    B -- Logged In --> C{Get Current Page};
    B -- Not Logged In --> R[Redirect to Login Page] --> Z[End];

    C --> D{Page Type?};
    D -- list --> E[initUsersList];
    D -- view --> F[initUserView];
    D -- edit --> G[initUserEdit];
    D -- create --> H[initUserCreate];
    D -- Other --> Z;

    subgraph "List Page Workflow"
        E --> E1[Show Loading];
        E1 --> E2{Check 'needsRefresh' Flag};
        E2 --> E3[Fetch Users Data];
        E3 --> E4[Call API GET /api/users];
        E4 --> E5{API Success?};
        E5 -- Yes --> E6[Process Data, Update Cache];
        E5 -- No --> E7[Handle Error, Use Cache or Test Data];
        E7 --> E6;
        E6 --> E8{Use Fetch Data or Cached Data?};
        E8 -- Fetch --> E9[Initialize DataTable with AJAX];
        E8 -- Cached --> E10[Initialize DataTable with Cached Data];
        E9 --> E11[Store Users Data];
        E10 --> E11;
        E11 --> E12[Setup Event Listeners];
        E12 --> E13[Hide Loading];
        E13 --> E14[Show Users Table];
        E14 --> E15{User Interaction?};
        E15 -- Filter --> E16[Apply Filter] --> E17[Redraw Table] --> E14;
        E15 -- Delete --> E18{Confirm Deletion?};
        E18 -- Yes --> E19[Call deleteUser(userId)];
        E19 --> E20[Call API DELETE /api/users/:id];
        E20 --> E21{API Success/Error?};
        E21 --> E22[Remove User, Update Cache];
        E22 --> E23[Reload Table];
        E23 --> E24[Show Success Message] --> E14;
        E18 -- No --> E14;
        E15 -- View/Edit --> E25[Go to View/Edit Page] --> Z;
        E15 -- Create --> E26[Go to Create Page] --> Z;
    end

    subgraph "View Page Workflow"
        F --> F1[Get userId from URL];
        F1 --> F2[Show Loading];
        F2 --> F3[Fetch User Details];
        F3 --> F4[Call API GET /api/users/:id];
        F4 --> F5{API Success?};
        F5 -- Yes --> F6[Process and Store User Data];
        F5 -- No --> F7[Handle Error, Use Cache];
        F7 --> F6;
        F6 --> F8[Fetch User Groups];
        F8 -- Needs Fetch --> F9[Call fetchGroupUsers];
        F9 --> F10{API Success?};
        F10 -- Yes --> F11[Store Groups Data];
        F10 -- No --> F12[Handle Error, Use Cache];
        F12 --> F11;
        F8 -- Data Available --> F11;
        F11 --> F13[Filter User Groups];
        F13 --> F14[Render User Details];
        F14 --> F15[Render User Teams];
        F15 --> F16[Setup Event Listeners];
        F16 --> F17[Hide Loading];
        F17 --> F18[Show User Details and Teams];
        F18 --> F19{User Interaction?};
        F19 -- Edit --> E25;
        F19 -- Delete --> E18V{Confirm Deletion?};
        E18V -- Yes --> E19V[Call deleteUser(userId)];
        E19V --> E20V[Call API DELETE /api/users/:id];
        E20V --> E21V{API Success/Error?};
        E21V --> E22V[Remove User, Update Cache];
        E22V --> E23V[Show Success Message];
        E23V --> E24V[Redirect to List Page] --> Z;
        E18V -- No --> F18;
        F19 -- Remove Team --> F20{Confirm Remove?};
        F20 -- Yes --> F21[Call removeUserFromTeam];
        F21 --> F22[Call API DELETE];
        F22 --> F23{API Success/Error?};
        F23 --> F24[Remove Link, Update Cache];
        F24 --> F25[Reload Teams Table];
        F25 --> F26[Show Success Message] --> F18;
        F20 -- No --> F18;
    end

    subgraph "Edit Page Workflow"
        G --> G1[Get userId from URL];
        G1 --> G2[Show Loading];
        G2 --> G3[Fetch Users Data];
        G3 --> G4{API Success?};
        G4 -- Yes --> G5[Store Data, Update Cache];
        G4 -- No --> G6[Handle Error, Use Cache];
        G6 --> G5;
        G5 --> G7[Find User in Data];
        G7 --> G8{User Exists?};
        G8 -- Yes --> G9{Check Permissions};
        G8 -- No --> G10[Show 'User Not Found'] --> G14[Hide Loading] --> Z;
        G9 -- Yes --> G11[Populate Edit Form];
        G9 -- No --> G12[Show 'Permission Denied'] --> G13[Redirect] --> G14 --> Z;
        G11 --> G15[Setup Event Listeners];
        G15 --> G14;
        G14 --> G16[Show Edit Form];
        G16 --> G17{Submit Form?};
        G17 -- Yes --> G18[Get Form Data];
        G18 --> G19[Call updateUser];
        G19 --> G20[Call API PUT /api/users/:id];
        G20 --> G21{API Success/Error?};
        G21 --> G22[Update User Data];
        G22 --> G23[Show Success Message];
        G23 --> G24[Redirect to View Page] --> Z;
        G17 -- No --> G16;
    end

    subgraph "Create Page Workflow"
        H --> H1{Check Permissions};
        H1 -- Yes --> H2[Setup Event Listeners];
        H1 -- No --> G12;
        H2 --> H3[Show Create Form];
        H3 --> H4{Submit Form?};
        H4 -- Yes --> H5[Get Form Data];
        H5 --> H6{Validate Data?};
        H6 -- Yes --> H7[Call createUser];
        H6 -- No --> H8[Show Validation Error] --> H3;
        H7 --> H9[Call API POST /api/users];
        H9 --> H10{API Success/Error?};
        H10 --> H11[Set Refresh Flag];
        H11 --> H12[Show Success Message];
        H12 --> H13[Redirect to List Page] --> Z;
        H4 -- No --> H3;
    end

    Z[End];
