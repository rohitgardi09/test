CREATE OR REPLACE VIEW view_refund_report as
    SELECT
        t.merchant_id                                         AS mid,
        DATE '1970-01-01' + ( r.created_date / 1000 ) / 86400 AS refund_booking_date,
        o.order_ref_number                                    AS merchant_order_number,
        o.customer_id                                         AS customer_id,
        o.currency_code                                       AS transaction_currency,
        o.order_amount                                        AS order_amount,
        t.debit_amt                                           AS debit_amount,
        '0'                                                   AS merchant_posting_amount,
        '0'                                                   AS gateway_posting_amount,
        o.sbi_order_ref_number                                AS sbi_order_ref_number,
        o.status                                              AS order_status,
        t.atrn_num                                            AS atrn_num,
        r.refund_type                                         AS refund_type,
        t.transaction_status                                  AS transaction_status,
        t.payment_success_date                                AS transaction_success_date_and_time,
        '0'                                                   AS refund_processed_date,
        r.refund_status                                       AS refund_status,
        t.pay_mode                                            AS paymode_name,
        t.channel_bank                                        AS bank_name,
        nvl(
            t.available_refund_amount, 0
        )                                                     AS amount_refunded,
        '0'                                                   AS further_refund_allowed,
        nvl(
            r.refund_amount - t.available_refund_amount, 0
        )                                                     AS pending_amount_for_refund,
        t.bank_reference_number,
        '0'                                                   AS settled_amount,
        nvl(
            r.refund_amount - t.available_refund_amount, 0
        )                                                     AS refund_amount,
        '0'                                                   AS chargeback_amount,
        r.remark                                              AS remark,
        r.arrn_num
    FROM
        merchant_order_payments_txn        t,
        merchant_orders_txn                o,
        merchant_order_hybrid_fee_dtls_txn h,
        refund_booking_txn                 r
    WHERE
        t.merchant_id = r.merchant_id
        AND t.atrn_num = r.atrn_num
        AND t.atrn_num = h.atrn_num
        AND t.order_ref_number = o.order_ref_number
        AND t.merchant_id = o.merchant_id
        AND t.sbi_order_ref_number = o.sbi_order_ref_number
        AND t.sbi_order_ref_number = r.sbi_order_ref_number;
