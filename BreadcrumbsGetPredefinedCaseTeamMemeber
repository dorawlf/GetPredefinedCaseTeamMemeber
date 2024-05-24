from simple_salesforce import Salesforce,format_soql
from datetime import datetime
import csv

session_id = input("Please Enter Access Token: ")

sf = Salesforce(instance_url='https://yourdomain.my.salesforce.com', session_id=session_id)

# Get all Case Team members (TeamID,MemberId)
data = sf.query("select id,MemberId,TeamTemplateId from CaseTeamTemplateMember where TeamTemplateId")
case_team_member_id_list =  [{'MemberId': item['MemberId'], 'TeamId': item['TeamTemplateId']} for item in data['records']]
user_id_list = []
for item in case_team_member_id_list:
    user_id_list.append(item['MemberId'])

# Get all users in Case Team
data = sf.query(format_soql("select id,name,IsActive from User where id IN {}",user_id_list))
user_list =  [{'Id': item['Id'], 'Name': item['Name'], 'Status': item['IsActive']} for item in data['records']]

# Get all contacts in Case Team
data = sf.query_all(format_soql("select id,name from Contact where id IN {}",user_id_list))
contact_list =  [{'Id': item['Id'], 'Name': item['Name'], 'Status': 'Contact'} for item in data['records']]

# Combine user and contact list as the member(id,name) list
member_list = user_list + contact_list

# Get all Case Teams
data = sf.query("select id,name from CaseTeamTemplate")
case_team_list =  [{'TeamId': item['Id'], 'TeamName': item['Name']} for item in data['records']]


# Create lookup dictionaries
team_dict = {team['TeamId']: team['TeamName'] for team in case_team_list}
member_dict = {member['Id']: member['Name'] for member in member_list}
status_dict = {member['Id']: member['Status'] for member in member_list}

case_team_member_list = []

# Loop to create the final list
for entry in case_team_member_id_list:
    team_id = entry['TeamId']
    member_id = entry['MemberId']

    case_team_member_list.append({
        'team_id': team_id,
        'team_name': team_dict[team_id],
        'member_id': member_id,
        'member_name': member_dict[member_id],
        'member_status':status_dict[member_id]
    })

case_team_member_list.sort(key=lambda x: x['team_name'])

# Export to CSV
keys = case_team_member_list[0].keys()

current_datetime = datetime.now()
formatted_datetime = current_datetime.strftime("%Y%m%d-%H%M%S")

with open('Case_Memeber-' + formatted_datetime + '.csv', 'w', newline='') as output_file:
    dict_writer = csv.DictWriter(output_file, keys)
    dict_writer.writeheader()
    dict_writer.writerows(case_team_member_list)
