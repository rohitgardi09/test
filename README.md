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
