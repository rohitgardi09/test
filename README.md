
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
