SELECT
    COUNT(*) AS totalTransactionCount,

    TO_CHAR(ROUND(SUM(order_amount), 2), 'FM9999990.00') AS totalOrderAmount,
    TO_CHAR(ROUND(SUM(refund_amount), 2), 'FM9999990.00') AS totalRefundAmount,
    TO_CHAR(ROUND(SUM(tax_amount), 2), 'FM9999990.00') AS totalTaxAmount,
    TO_CHAR(ROUND(SUM(net_settlement_amount), 2), 'FM9999990.00') AS totalNetSettlementAmount,
    TO_CHAR(ROUND(SUM(settled_amount), 2), 'FM9999990.00') AS totalSettledAmount,
    TO_CHAR(ROUND(SUM(pending_settlement_amount), 2), 'FM9999990.00') AS totalPendingSettlementAmount

FROM transactions
WHERE status = 'SUCCESS';
