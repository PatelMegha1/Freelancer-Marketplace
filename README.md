# ServiceNow Freelancer-Marketplace
A ServiceNow custom application that allows **clients** to post projects and automatically matches them with the **top 3 freelancers** based on required skills. Built with custom tables, dynamic matching logic, and related lists for real-time visibility.

---

## 📦 Features

- Clients can post new projects with budget, deadline, and required skills.
- Freelancers create profiles with skills and availability.
- Automatic freelancer matching logic using a Business Rule.
- Displays the top 3 best-fit freelancers per project with match scores and ordering.
- Admin view of all matches with sortable scores.

---

## 🗃️ Database Tables

| Table Name                      | Description                                                                 |
|-------------------------------|-----------------------------------------------------------------------------|
| `Client`                       | Stores client profile information                                           |
| `Freelancer`                   | Stores freelancer details including skills and availability                 |
| `Project`                      | Stores project details like title, description, budget, skills, and deadline |
| `Project Freelancer Match`     | Stores the top 3 matched freelancers per project with match score and order |

---

## 📋 Project Table Fields

| Field          | Type           | Description                          |
|---------------|----------------|--------------------------------------|
| Title          | String         | Project title                        |
| Description    | String         | Project description                  |
| Budget         | Decimal        | Estimated project budget             |
| Deadline       | Date           | Project completion deadline          |
| Client Name    | Reference      | Reference to the Client table        |
| Required Skills| Multi-select List | Predefined list of skill choices (e.g., Java, NodeJS, UI/UX) |

---

## 👤 Freelancer Table Fields

| Field         | Type           | Description                              |
|--------------|----------------|------------------------------------------|
| Name          | String         | Freelancer's full name                   |
| Email         | Email          | Contact email                            |
| Availability  | Choice         | Part-time / Full-time / Not available    |
| Skills        | Multi-select List | Freelancer's skill set from predefined options |

---

## 🔄 Automatic Matching Logic

A Business Rule is triggered after a project is created or updated. It compares the project's required skills with all freelancers' skills and selects the **top 3 best matches** based on skill overlap.

### When it runs:
- **After**: Insert or Update
- **Advanced**: Enabled

---

## 🤖 Business Rule Summary

```javascript
(function executeRule(current, previous /*null when async*/) {

    if (!current.required_skills) return;

    var projectSkills = current.required_skills.split(',');
    var freelancerMatches = [];

    var freelancer = new GlideRecord('x_678463_freelan_0_freelancer');
    freelancer.query();

    while (freelancer.next()) {
        var freelancerSkills = freelancer.skills ? freelancer.skills.split(',') : [];
        var matchCount = 0;

        for (var i = 0; i < projectSkills.length; i++) {
            if (freelancerSkills.indexOf(projectSkills[i]) !== -1) {
                matchCount++;
            }
        }

        if (matchCount > 0) {
            freelancerMatches.push({
                id: freelancer.sys_id.toString(),
                score: matchCount
            });
        }
    }

    freelancerMatches.sort(function(a, b) {
        return b.score - a.score;
    });

    var oldMatches = new GlideRecord('x_678463_freelan_0_project_freelancer_match');
    oldMatches.addQuery('project', current.sys_id);
    oldMatches.deleteMultiple();

    for (var j = 0; j < Math.min(3, freelancerMatches.length); j++) {
        var match = new GlideRecord('x_678463_freelan_0_project_freelancer_match');
        match.initialize();
        match.project = current.sys_id;
        match.freelancer = freelancerMatches[j].id;
        match.match_score = freelancerMatches[j].score;
        match.order = j + 1;
        match.insert();
    }

})(current, previous);
