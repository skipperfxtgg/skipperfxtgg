from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///gonow.db'
db = SQLAlchemy(app)

# Define the Rider model
class Rider(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), nullable=False)
    location = db.Column(db.String(100), nullable=False)

# Define the Driver model
class Driver(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), nullable=False)
    car_model = db.Column(db.String(50), nullable=False)
    location = db.Column(db.String(100), nullable=False)
    is_available = db.Column(db.Boolean, default=True)

# Define the Ride model
class Ride(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    rider_id = db.Column(db.Integer, db.ForeignKey('rider.id'), nullable=False)
    driver_id = db.Column(db.Integer, db.ForeignKey('driver.id'), nullable=False)
    start_location = db.Column(db.String(100), nullable=False)
    end_location = db.Column(db.String(100), nullable=False)
    status = db.Column(db.String(50), nullable=False)  # e.g., "requested", "ongoing", "completed"

# Endpoint to request a ride
@app.route('/ride/request', methods=['POST'])
def request_ride():
    rider_id = request.json['rider_id']
    start_location = request.json['start_location']
    end_location = request.json['end_location']
    
    # Find an available driver
    driver = Driver.query.filter_by(is_available=True).first()
    if driver:
        new_ride = Ride(rider_id=rider_id, driver_id=driver.id,
                        start_location=start_location, end_location=end_location,
                        status='requested')
        db.session.add(new_ride)
        db.session.commit()
        return jsonify({'message': 'Ride requested successfully', 'ride_id': new_ride.id}), 200
    else:
        return jsonify({'message': 'No available drivers'}), 404

# Endpoint to update ride status
@app.route('/ride/update/<int:ride_id>', methods=['PUT'])
def update_ride(ride_id):
    ride = Ride.query.get_or_404(ride_id)
    ride.status = request.json['status']
    db.session.commit()
    return jsonify({'message': 'Ride status updated successfully'}), 200

if __name__ == '__main__':
    db.create_all()
    app.run(debug=True)
    
