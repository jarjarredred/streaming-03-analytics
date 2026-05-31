# Streaming Data

This site provides documentation for this project.
Use the navigation to explore module-specific materials.

# Custom Project Documentation: Kafka Analytics Consumer

## Custom Project

### Dataset
Describe the dataset used by your Kafka producer.

* **Dataset File**: `data/sales.csv` (used by the producer to feed Kafka).
* **Record Type**: E-commerce sales transactions.
* **Included Fields**: `order_id`, `customer_id`, `product_id`, `quantity`, `unit_price`, `currency_code`, `region_id`, `payment_method`, `device_type`, and `datetime`.
* **Modification**: Used the standard sales dataset but enriched it dynamically during consumption.
* **Reference Datasets**:
    * `regions.csv`: Used to map `region_id` to tax rates.
    * `products.csv`: Used for product validation.

---

### Data Contract
Describe the rules your messages must follow.

* **Required Fields**: `order_id`, `product_id`, `quantity`, `unit_price`, and `region_id`.
* **Optional Fields**: `customer_note`, `discount_code`.
* **Validation Rules**: `region_id` must exist in the `region_lookup` (derived from `regions.csv`).
* **Validity Criteria**: A message is valid if all required fields are present and data types (like quantity) are numeric; it is invalid if the `region_id` is missing, which prevents tax calculation.

---

### Kafka Messages
Describe the messages sent through Kafka.

* **Producer Content**: JSON-serialized sales records.
* **Kafka Topic**: `streaming-03-analytics-case`.
* **Message Key**: `region_id` (ensures all sales from the same region are routed to the same partition for consistent processing).
* **Changes**: No changes to the raw fields sent by the producer, but metadata like `_kafka_offset` and `_kafka_partition` were added by the broker for tracking.

---

### Consumer Validation
Describe how your consumer checks each message.

* **Validation Checks**: The consumer verifies the existence of all `SALES_REQUIRED_FIELDS` before any calculation occurs.
* **Accepted Messages**: Messages are enriched with derived financial fields and appended to the output CSV.
* **Rejected Messages**: If validation fails, the consumer logs a warning, increments `skipped_count`, and continues to the next message without saving it to disk.
* **Data Protection**: This layer prevents "garbage" data (like an order with a null price) from corrupting the running averages or global totals.

---

### Data Engineering and Enrichment
Describe what your consumer computes or adds.

* **Derived Fields**: `subtotal`, `tax_amount`, and `total`.
* **Reference Data**: The `region_lookup` dictionary (loaded from `regions.csv`).
* **Calculations**:
    * **Subtotal**: Subtotal = Quantity × Unit Price
    * **Tax Amount**: Tax Amount = Subtotal × Tax Rate
    * **Total**: Total = Subtotal + Tax Amount
* **Specific Changes**: Maintained the enrichment logic but modified the logger to track a mid-process `running_total` and specific sale counts to verify state management.

---

### Streaming Analytics
Describe the running summaries created as messages arrive.

* **Values Summarized**: Global sales metrics and **Stateful Regional Sales Totals**.
* **Tracked Statistics**:
    * **Global**: Total count, sum, mean, minimum, and maximum.
    * **Custom**: A running cumulative sum for each unique `region_id` stored in a dictionary.
* **Dynamic Behavior**: As messages arrived, the `regional_totals` state updated specific buckets (like `US-TX`) independently of other regions.

---

### Experiments
Describe the small technical changes you made.

* **Phase 4 (Logic Modification)**: Updated `process_message` to log specific transaction counts (e.g., `Processed Sale #2`) and report the `running_total` immediately after calculation.
* **Phase 5 (Stateful Application)**: Introduced a **Stateful Dictionary** (`regional_totals`) that persists across the consumer loop, allowing for aggregated reporting that survives beyond the scope of a single message.

---

### Results
Describe what happened when you ran the producer and consumer.

* **Production**: Successful; 3 messages were sent to the topic.
* **Consumption**: Successful; all 3 messages were retrieved from the beginning of the topic using the `OFFSET_BEGINNING` reset.
* **Final Counts**: 3 accepted, 0 rejected/skipped.
* **Output CSV**: Verified in `data\output\consumed_sales_jarred2.csv` with all derived fields populated.
* **Logs**: Terminal output confirmed the stateful changes:
    * `Updated US-TX total: $194.82` (First Sale)
    * `Updated US-TX total: $248.93` (Second Sale - aggregated correctly)
    * `Updated CA-QC total: $63.22` (Third Sale - new bucket created)

---

### Interpretation
Explain what the validation and analytics workflow showed you.

* **Evolution from Example**: The original example was stateless regarding categories; my modification turned the consumer into a real-time analytics engine capable of geographic segmentation.
* **Validation Insight**: Validating before enrichment is crucial; it ensures the "contract" is met before performing expensive or crash-prone calculations.
* **Enrichment Insight**: Moving calculations to the consumer level allows for "Business Intelligence on the fly," transforming raw logs into financial data instantly.
* **Organizational Value**: Real-time regional summaries allow an organization to monitor geographic performance instantly, rather than waiting for a daily batch report.
* **Business Intelligence**: The run successfully identified `US-TX` as a high-performing region compared to `CA-QC`, providing the data needed for immediate logistical decisions.

## How-To Guide

Many instructions are common to all our projects.

See
[⭐ **Workflow: Apply Example**](https://denisecase.github.io/pro-analytics-02/workflow-b-apply-example-project/)
to get these projects running on your machine.

## Project Documentation Pages (docs/)

- **Home** - this documentation landing page
- **Project Instructions** - instructions specific to this module
- **Your Files** - how to copy from examples and make them yours
- **Glossary** - project terms and concepts
- **API** - autogenerated look at the code interface
