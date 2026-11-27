# Keepr app

Next.js browser app, owned by C. Create it here with:

```bash
npx create-next-app@latest .
npm install onnxruntime-web @mediapipe/tasks-vision
```

It loads `models/<run>_int8.onnx`, groups photos into bursts by embedding similarity, scores
every pair in a burst, and shows a "Best" badge or "Not sure" when calibrated confidence is
below the threshold.
