# MUYKO E-commerce Implementation Plan

## Recommended Solution: Nebulix + Custom Fulfillment Integration

### Why This Approach:
- ✅ **Fast Implementation**: 2-3 days vs 2-3 weeks
- ✅ **Low Maintenance**: No complex backend required
- ✅ **Fulfillment Ready**: Built-in API and webhook support
- ✅ **Cost Effective**: No monthly Shopify fees
- ✅ **MUYKO Branding**: Easily customizable to match your design

## Phase 1: E-commerce Setup (Day 1-2)

### 1. Install Nebulix Template
```bash
# Clone and integrate Nebulix
git clone https://github.com/unfolding-io/nebulix.git temp-nebulix
cp -r temp-nebulix/src/components/shop/* src/components/
cp -r temp-nebulix/src/pages/shop/* src/pages/
```

### 2. Configure Snipcart
```javascript
// src/components/SnipcartConfig.astro
window.SnipcartSettings = {
  publicApiKey: "YOUR_SNIPCART_API_KEY",
  loadStrategy: "on-user-interaction",
  modalStyle: "side",
  currency: "usd",
  templatesUrl: "/snipcart-templates.html"
};
```

### 3. Product Setup
```typescript
// src/data/products.ts
export const products = [
  {
    id: "muyko-product-1",
    name: "MUYKO Product 1",
    price: 99.00,
    image: "/images/product-1.jpg",
    description: "Premium MUYKO product description",
    sku: "MUYKO-001",
    inventory: 100
  },
  {
    id: "muyko-product-2", 
    name: "MUYKO Product 2",
    price: 149.00,
    image: "/images/product-2.jpg", 
    description: "Premium MUYKO product description",
    sku: "MUYKO-002",
    inventory: 75
  }
];
```

## Phase 2: Fulfillment Integration (Day 3)

### 1. Webhook Endpoint
```typescript
// src/pages/api/fulfillment-webhook.ts
export async function POST({ request }) {
  const order = await request.json();
  
  // Validate Snipcart webhook
  if (!validateSnipcartSignature(request)) {
    return new Response('Unauthorized', { status: 401 });
  }
  
  // Send to fulfillment house
  const fulfillmentOrder = {
    external_order_id: order.invoice.number,
    customer: {
      name: `${order.invoice.billingAddress.name}`,
      email: order.email,
      phone: order.invoice.billingAddress.phone
    },
    shipping_address: {
      name: order.invoice.shippingAddress.name,
      address1: order.invoice.shippingAddress.address1,
      address2: order.invoice.shippingAddress.address2,
      city: order.invoice.shippingAddress.city,
      state: order.invoice.shippingAddress.province,
      zip: order.invoice.shippingAddress.postalCode,
      country: order.invoice.shippingAddress.country
    },
    line_items: order.invoice.items.map(item => ({
      sku: item.id,
      quantity: item.quantity,
      name: item.name,
      price: item.price
    }))
  };
  
  // Send to your fulfillment provider
  await sendToFulfillmentHouse(fulfillmentOrder);
  
  return new Response('OK');
}
```

### 2. Fulfillment Provider Integration
```typescript
// Common fulfillment providers integration examples

// ShipStation API
async function sendToShipStation(order) {
  return fetch('https://ssapi.shipstation.com/orders/createorder', {
    method: 'POST',
    headers: {
      'Authorization': `Basic ${btoa(SS_API_KEY + ':' + SS_API_SECRET)}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(order)
  });
}

// ShipBob API  
async function sendToShipBob(order) {
  return fetch('https://api.shipbob.com/1.0/order', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${SHIPBOB_TOKEN}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(order)
  });
}

// Generic fulfillment house API
async function sendToFulfillmentHouse(order) {
  return fetch('https://your-fulfillment-api.com/orders', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer YOUR_API_KEY',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(order)
  });
}
```

### 3. Tracking Updates
```typescript
// src/pages/api/tracking-update.ts
export async function POST({ request }) {
  const { order_id, tracking_number, carrier, status } = await request.json();
  
  // Update order in Snipcart
  await fetch(`https://app.snipcart.com/api/orders/${order_id}`, {
    method: 'PUT',
    headers: {
      'Authorization': `Basic ${btoa(SNIPCART_SECRET_KEY + ':')}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      trackingNumber: tracking_number,
      trackingUrl: `https://${carrier}.com/track/${tracking_number}`
    })
  });
  
  // Send customer notification email
  await sendTrackingEmail(order_id, tracking_number);
  
  return new Response('OK');
}
```

## Phase 3: MUYKO Branding Integration

### 1. Customize Cart Design
```astro
<!-- src/components/MuykoCart.astro -->
<style>
#snipcart {
  --color-primary: #your-brand-color;
  --color-secondary: #your-secondary-color;
  --font-family: 'Your Brand Font', sans-serif;
}
</style>
```

### 2. Product Components
```astro
<!-- src/components/MuykoProductCard.astro -->
---
interface Props {
  product: Product;
}
const { product } = Astro.props;
---

<article class="muyko-product-card">
  <img src={product.image} alt={product.name} />
  <h3>{product.name}</h3>
  <p class="price">${product.price}</p>
  <button 
    class="snipcart-add-item btn-primary"
    data-item-id={product.id}
    data-item-price={product.price}
    data-item-url={`/products/${product.id}`}
    data-item-description={product.description}
    data-item-image={product.image}
    data-item-name={product.name}
  >
    Add to Cart
  </button>
</article>
```

## Fulfillment House Requirements

### What You'll Need:
1. **API Credentials** from your fulfillment provider
2. **Product SKUs** mapped to fulfillment house inventory
3. **Webhook URLs** configured in fulfillment system
4. **Shipping Rules** and cost calculations

### Supported Fulfillment Providers:
- ✅ **ShipStation** (Most popular)
- ✅ **ShipBob** (Warehousing + fulfillment)
- ✅ **Fulfillment by Amazon (FBA)**
- ✅ **Red Stag Fulfillment**
- ✅ **Easyship**
- ✅ **Custom API** (Any provider with REST API)

## Timeline & Costs

### Development Time:
- **Template Integration**: 1 day
- **Product Setup**: 4 hours  
- **Fulfillment Integration**: 1 day
- **Testing & Launch**: 4 hours
- **Total**: 2-3 days

### Ongoing Costs:
- **Snipcart**: 2% transaction fee (no monthly fee)
- **Fulfillment**: Per-order fees (varies by provider)
- **Hosting**: Current (no change)

## Next Steps

1. **Choose fulfillment provider** and get API credentials
2. **Set up Snipcart account** and get API keys
3. **Implement template** with MUYKO branding
4. **Configure fulfillment integration**
5. **Test order flow** end-to-end
6. **Launch** with real products

Would you like me to start implementing this solution?
