# Addis Eats — Live Delivery Tracking

Next.js App Router demo for an order delivery tracking workflow with a client-only map, courier and destination markers, server-sent events (SSE), shared date/currency formatters, and robust CSV export.

## Requirements

- Node.js 20.9+
- npm

## Run

```bash
npm install
cp .env.example .env.local
npm run dev
```

On Windows PowerShell:

```powershell
Copy-Item .env.example .env.local
npm run dev
```

Visit:
- `http://localhost:3000` — order overview
- `http://localhost:3000/delivery/AE-2048` — live delivery tracking
- `http://localhost:3000/api/orders/AE-2048/stream` — raw SSE stream (inspect in a browser or curl)

## Requirement checklist

1. **Client-only map:** `components/DeliveryMap.tsx` uses `next/dynamic` with `ssr: false` and a 360px sized loading placeholder. The map canvas renders only on the client.
2. **Courier + destination markers:** the map shows both labeled markers and a route line. The adjacent map description repeats both locations with latitude/longitude, making the locations available as text to assistive technology. The schematic automatically keeps both markers in the map frame.
3. **SSE route:** `app/api/orders/[id]/stream/route.ts` returns `text/event-stream`, uses correct `data: ...\\n\\n` event framing, includes heartbeat comments, disables caching/buffering, and closes after delivery.
4. **EventSource client:** `DeliveryTracker` seeds its state from the server-provided order, connects with `EventSource`, merges updates, and calls `source.close()` in the effect cleanup.
5. **Connection indicator + deliberate close:** connection states include connecting, connected, interrupted, and closed. On a `Delivered` event, the client explicitly closes EventSource and shows the closed state.
6. **Shared `lib/format.js`:** exports `etb`, `when`, and `relative`. The dashboard, order table, and tracker use these shared formatters.
7. **CSV export:** the export button downloads UTF-8 CSV with BOM and CRLF rows. The RFC 4180 escape rule wraps values containing commas, quotation marks, or line breaks in quotes and doubles internal quotation marks.
8. **Awkward value tests:** the demo records deliberately contain a customer with a comma and quotation mark and a dish with a comma and quotation mark. Run `npm run test:csv` to test comma, quote, and newline escaping.

## Verify SSE

Start the server and visit `/api/orders/AE-2048/stream`. Each event is framed with an optional `event: status` line, a `data: {JSON}` line, and a blank line. The demo advances the courier position and sends `Delivered` after five update ticks. The client closes the stream when delivery is complete.

## CSV behavior

Click **Export CSV** on the delivery page. Open the file with a text editor to verify:
- `Mekdes Tadesse, "Mimi"` becomes `"Mekdes Tadesse, ""Mimi"""`.
- `Family combo, "special"` becomes `"Family combo, ""special"""`.

This quoting prevents commas or quotes inside a cell from shifting the CSV columns.

## Notes

- The map is an accessible schematic demo, not a live geographic map tile provider. Coordinates are example Addis Ababa locations. Replace the canvas with a map provider if true roads and live GPS are needed.
- Orders are deterministic in-memory demo data. Replace `lib/orders.js` with database-backed records and authorize the SSE route before using real customer information.
- The stream is a demo simulation; in production, connect to real order status updates and use a shared event broker if the app runs on multiple instances.
