from flask import Flask, request, jsonify, send_from_directory
from predictor import FIFAPredictor

app = Flask(__name__, static_folder="static")

@app.route('/')
def home():
    return send_from_directory('static', 'index.html')

@app.route('/predict', methods=['POST'])
def predict():
    try:
        data = request.json
        scores = [tuple(map(int, s.strip().split('-'))) for s in data['scores']]
        predictor = FIFAPredictor(data['equipe_dom'], data['equipe_ext'], scores, methode=data['methode'])
        
        predictions = predictor.predire_scores()
        btts = predictor.equipe_marque()
        total = predictor.total_buts_moyen()
        
        return jsonify({
            "predictions": [{"score": f"{a}-{b}", "proba": f"{p*100:.2f}%"} for (a, b), p in predictions],
            "btts": {
                "equipe_A": btts['equipe_A'],
                "equipe_B": btts['equipe_B'],
                "les_deux": btts['les_deux'],
                "proba": f"{btts['proba']*100:.2f}%"
            },
            "total_buts": f"{total:.2f}"
        })
    except Exception as e:
        return jsonify({"error": str(e)}), 400

if __name__ == '__main__':
    app.run(debug=True)
