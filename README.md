
I checked the views related to this validation.

Currently, the validation is happening through MERCHANT_PAYMODE_VIEW, which mainly contains paymode and payment gateway configuration details coming from the "aggregatormerchantpaymode" table.

However, as per the QA expectation, the validation should be done from MERCHANT_INFO_VIEW. This view already combines merchant-related information by joining multiple tables, including Aggmerchantchargedetails (C).

Since the required charge-related details are coming from Aggmerchantchargedetails (C) and are mapped in the form formAggMerchantChargedDetailsC, the validation should ideally be performed from MERCHANT_INFO_VIEW instead of MERCHANT_PAYMODE_VIEW.

Also, in the MERCHANT_INFO_VIEW, the data is already available paymode-wise, so handling the validation from this view would align with the expected data structure and QA requirement.

Please let me know if any additional changes are required from my side.

























/**
 * Updates the report status when a ReportingException occurs during
 * report generation.
 *
 * If the exception message indicates that no records were found,
 * the report status is set to NO_RECORD_FOUND. Otherwise, the
 * report status is considered as GENERATION_FAILED.
 *
 * This method also logs the appropriate error details along with
 * the report information to help in debugging and monitoring.
 *
 * @param reportManagementDto contains report details such as report id and report name
 * @param e the ReportingException thrown during report generation
 */




****************/**
 * Registers a custom Jackson {@link SimpleModule} that configures a
 * {@link TwoDecimalSerializer} for numeric types.
 *
 * This module ensures that all numeric values such as {@link BigDecimal},
 * {@link Double}, {@code double}, {@link Float}, and {@code float}
 * are serialized with exactly two decimal places during JSON serialization.
 *
 * This helps maintain consistent numeric formatting across the application
 * when converting Java objects into JSON responses.
 *
 * @return A Jackson {@link com.fasterxml.jackson.databind.Module} configured
 *         to serialize numeric values with two decimal precision.
 */


 






/**
 * Custom Jackson serializer used to format numeric values
 * with exactly two decimal places during JSON serialization.
 *
 * This serializer supports {@link BigDecimal}, {@link Double},
 * {@link Float}, and their primitive equivalents.
 *
 * If the value is {@code null}, it writes a JSON null.
 * If the value type is unsupported, it falls back to default serialization.
 *
 * Rounding is applied using {@link RoundingMode#HALF_UP}
 * to ensure standard financial rounding behavior.
 */
public class TwoDecimalSerializer extends JsonSerializer<Object> {

    /**
     * Serializes numeric values into JSON with two decimal precision.
     *
     * @param value        the value to serialize
     * @param gen          JSON generator used to write output
     * @param serializers serializer provider
     * @throws IOException if an I/O error occurs during writing
     */







SimpleModule module = new SimpleModule();
        module.addSerializer(
                BigDecimal.class,
                new TwoDecimalBigDecimalSerializer()
        );


-----------------------------------------------------------------------------------------------------------------------------

import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;

import java.io.IOException;
import java.math.BigDecimal;
import java.math.RoundingMode;

public class TwoDecimalBigDecimalSerializer
        extends JsonSerializer<BigDecimal> {

    @Override
    public void serialize(BigDecimal value,
                          JsonGenerator gen,
                          SerializerProvider serializers)
            throws IOException {

        // if (StringUtils.isEmpty(value)) {   // BigDecimal साठी invalid
        //     gen.writeNumber(
        //         BigDecimal.ZERO.setScale(2, RoundingMode.HALF_UP)
        //     );
        //     return;
        // }

        if (value == null) {
            gen.writeNumber(
                BigDecimal.ZERO.setScale(2, RoundingMode.HALF_UP)
            );
            return;
        }

        gen.writeNumber(
            value.setScale(2, RoundingMode.HALF_UP)
        );
    }
}
