require("dotenv").config();
const express = require("express");
const cors = require("cors");
const crypto = require("crypto");
const Razorpay = require("razorpay");

const app = express();
app.use(cors());
app.use(express.json());

const razorpay = new Razorpay({
  key_id: process.env.RAZORPAY_KEY_ID,
  key_secret: process.env.RAZORPAY_KEY_SECRET,
});

const PLANS = {
  creator: { amount: 39900, description: "Reel Creator Plan — monthly" },
  credits: { amount: 1500, description: "Reel Credits — 1 video" },
};

app.post("/create-order", async (req, res) => {
  try {
    const { plan } = req.body;
    const selectedPlan = PLANS[plan];

    if (!selectedPlan) {
      return res.status(400).json({ error: "Invalid plan selected" });
    }

    const order = await razorpay.orders.create({
      amount: selectedPlan.amount,
      currency: "INR",
      receipt: "receipt_" + Date.now(),
    });

    res.json({
      order_id: order.id,
      amount: order.amount,
      currency: order.currency,
      key_id: process.env.RAZORPAY_KEY_ID,
      description: selectedPlan.description,
    });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: "Order create nahi ho paya" });
  }
});

app.post("/verify-payment", (req, res) => {
  const { razorpay_order_id, razorpay_payment_id, razorpay_signature, user_id, plan } = req.body;

  const generatedSignature = crypto
    .createHmac("sha256", process.env.RAZORPAY_KEY_SECRET)
    .update(razorpay_order_id + "|" + razorpay_payment_id)
    .digest("hex");

  const isValid = generatedSignature === razorpay_signature;

  if (!isValid) {
    return res.status(400).json({ success: false, message: "Signature match nahi hui — payment fake ho sakta hai" });
  }

  console.log(`Payment verified for user ${user_id}, plan: ${plan}`);

  res.json({ success: true, message: "Payment verify ho gaya, plan activate kar diya gaya" });
});

app.get("/", (req, res) => {
  res.send("Reel backend chal raha hai");
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server chal raha hai: http://localhost:${PORT}`);
});
