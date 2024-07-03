# Digitalization-of-the-Hospitality-Process
import csv
from flask import Flask, request, jsonify

app = Flask(__name__)

# Load hostel rooms from CSV file
hostel_rooms = {}
with open('hostel_rooms.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        hostel_name = row['Hostel Name']
        room_number = int(row['Room Number'])
        capacity = int(row['Capacity'])
        gender = row['Gender']
        if hostel_name not in hostel_rooms:
            hostel_rooms[hostel_name] = []
        hostel_rooms[hostel_name].append({'room_number': room_number, 'capacity': capacity, 'gender': gender})

# Load groups from CSV file
groups = {}
with open('groups.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        group_id = int(row['Group ID'])
        member_count = int(row['Members'])
        gender = row['Gender']
        groups[group_id] = {'member_count': member_count, 'gender': gender}

@app.route('/upload', methods=['POST'])
def upload_files():
    # Parse CSV files from request
    group_csv = request.files['group_csv']
    hostel_csv = request.files['hostel_csv']
    groups_data = parse_csv(group_csv)
    hostel_data = parse_csv(hostel_csv)

    # Allocate rooms
    allocation = allocate_rooms(groups_data, hostel_data)

    # Return allocation results
    return jsonify(allocation)

def allocate_rooms(groups, hostel_rooms):
    allocation = {}
    for group_id, group in groups.items():
        gender = group['gender']
        member_count = group['member_count']
        matching_rooms = [room for hostel, rooms in hostel_rooms.items() for room in rooms if room['gender'] == gender]
        matching_rooms.sort(key=lambda x: x['capacity'], reverse=True)
        allocated_rooms = []
        for room in matching_rooms:
            if room['capacity'] >= member_count:
                allocated_rooms
