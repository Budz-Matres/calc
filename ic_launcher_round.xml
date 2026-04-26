package com.example.calc

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.foundation.verticalScroll
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.TrendingUp
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.toArgb
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.unit.dp
import androidx.compose.ui.viewinterop.AndroidView
import com.example.calc.ui.theme.CalcTheme
import com.github.mikephil.charting.charts.LineChart
import com.github.mikephil.charting.components.XAxis
import com.github.mikephil.charting.data.Entry
import com.github.mikephil.charting.data.LineData
import com.github.mikephil.charting.data.LineDataSet
import com.github.mikephil.charting.formatter.ValueFormatter
import java.text.NumberFormat
import java.util.Locale
import kotlin.math.pow
import kotlin.math.roundToInt

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            CalcTheme {
                Surface(modifier = Modifier.fillMaxSize(), color = MaterialTheme.colorScheme.background) {
                    CompoundInterestScreen()
                }
            }
        }
    }
}

// ── Data ──────────────────────────────────────────────────────────────────────

enum class CompoundFrequency(val label: String, val n: Int) {
    DAILY("Daily", 365),
    WEEKLY("Weekly", 52),
    MONTHLY("Monthly", 12),
    QUARTERLY("Quarterly", 4),
    SEMI_ANNUALLY("Semi-Annually", 2),
    ANNUALLY("Annually", 1)
}

data class Result(
    val total: Double,
    val principal: Double,
    val contributions: Double,
    val interest: Double,
    val yearly: List<Triple<Int, Double, Double>>  // year, invested, total
)

// ── Math ──────────────────────────────────────────────────────────────────────

fun compound(p: Double, rPct: Double, freq: CompoundFrequency, t: Double, pmt: Double): Result {
    val r = rPct / 100.0
    val n = freq.n.toDouble()

    fun principalAt(yr: Double) = p * (1 + r / n).pow(n * yr)

    fun contributionAt(yr: Double): Double {
        if (pmt == 0.0) return 0.0
        val months = yr * 12
        return if (r == 0.0) pmt * months
        else pmt * ((1 + r / 12.0).pow(months) - 1) / (r / 12.0)
    }

    val totalA = principalAt(t) + contributionAt(t)
    val totalContrib = pmt * 12 * t
    val yearly = (0..t.toInt()).map { yr ->
        val invested = p + pmt * 12 * yr
        val value = principalAt(yr.toDouble()) + contributionAt(yr.toDouble())
        Triple(yr, invested, value)
    }
    return Result(totalA, p, totalContrib, totalA - p - totalContrib, yearly)
}

// ── Screen ────────────────────────────────────────────────────────────────────

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun CompoundInterestScreen() {
    val currency = remember { NumberFormat.getCurrencyInstance(Locale.US) }

    var principal by remember { mutableStateOf("10000") }
    var rate by remember { mutableStateOf("5") }
    var years by remember { mutableFloatStateOf(10f) }
    var monthly by remember { mutableStateOf("0") }
    var freq by remember { mutableStateOf(CompoundFrequency.MONTHLY) }
    var freqOpen by remember { mutableStateOf(false) }
    var whatIfExtraYearsYears by remember { mutableFloatStateOf(0f) }

    val result by remember(principal, rate, years, monthly, freq) {
        derivedStateOf {
            val p = principal.toDoubleOrNull() ?: return@derivedStateOf null
            val r = rate.toDoubleOrNull() ?: return@derivedStateOf null
            val pmt = monthly.toDoubleOrNull() ?: 0.0
            if (p < 0 || r < 0 || years <= 0f) return@derivedStateOf null
            compound(p, r, freq, years.toDouble(), pmt)
        }
    }

    val whatIfResult by remember(principal, rate, years, monthly, freq, whatIfExtraYears) {
        derivedStateOf {
            if (whatIfExtraYears == 0f) return@derivedStateOf null
            val p = principal.toDoubleOrNull() ?: return@derivedStateOf null
            val r = rate.toDoubleOrNull() ?: return@derivedStateOf null
            val pmt = monthly.toDoubleOrNull() ?: 0.0
            compound(p, r, freq, years.toDouble() + whatIfExtraYears, pmt)
        }
    }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .verticalScroll(rememberScrollState())
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        // ── Header ────────────────────────────────────────────────────────────
        Column(modifier = Modifier.padding(vertical = 8.dp)) {
            Text(
                "Compound Interest",
                style = MaterialTheme.typography.headlineMedium,
                color = MaterialTheme.colorScheme.primary
            )
            Text(
                "Calculator",
                style = MaterialTheme.typography.headlineMedium,
                fontWeight = FontWeight.ExtraBold,
                color = MaterialTheme.colorScheme.primary
            )
            Spacer(Modifier.height(4.dp))
            Text(
                "A = P(1 + r/n)ⁿᵗ",
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
        }

        // ── Inputs ────────────────────────────────────────────────────────────
        Card(modifier = Modifier.fillMaxWidth(), shape = RoundedCornerShape(20.dp)) {
            Column(modifier = Modifier.padding(16.dp), verticalArrangement = Arrangement.spacedBy(12.dp)) {
                Text("Inputs", style = MaterialTheme.typography.titleMedium)

                Row(modifier = Modifier.fillMaxWidth(), horizontalArrangement = Arrangement.spacedBy(12.dp)) {
                    MoneyField(
                        value = principal, label = "Principal (P)",
                        prefix = "$", onValueChange = { principal = it },
                        modifier = Modifier.weight(1f)
                    )
                    MoneyField(
                        value = rate, label = "Annual Rate (R)",
                        suffix = "%", onValueChange = { rate = it },
                        modifier = Modifier.weight(1f)
                    )
                }

                MoneyField(
                    value = monthly, label = "Monthly Contribution",
                    prefix = "$", onValueChange = { monthly = it },
                    modifier = Modifier.fillMaxWidth()
                )

                // Frequency dropdown
                ExposedDropdownMenuBox(expanded = freqOpen, onExpandedChange = { freqOpen = it }) {
                    OutlinedTextField(
                        value = freq.label,
                        onValueChange = {},
                        readOnly = true,
                        label = { Text("Compound Frequency (n)") },
                        trailingIcon = { ExposedDropdownMenuDefaults.TrailingIcon(expanded = freqOpen) },
                        modifier = Modifier
                            .fillMaxWidth()
                            .menuAnchor()
                    )
                    ExposedDropdownMenu(expanded = freqOpen, onDismissRequest = { freqOpen = false }) {
                        CompoundFrequency.entries.forEach { f ->
                            DropdownMenuItem(
                                text = { Text("${f.label} (${f.n}x/yr)") },
                                onClick = { freq = f; freqOpen = false }
                            )
                        }
                    }
                }

                // Time slider
                Column {
                    Row(Modifier.fillMaxWidth(), horizontalArrangement = Arrangement.SpaceBetween) {
                        Text("Time (t)", style = MaterialTheme.typography.bodyMedium)
                        Text(
                            "${years.roundToInt()} years",
                            fontWeight = FontWeight.Bold,
                            color = MaterialTheme.colorScheme.primary
                        )
                    }
                    Slider(
                        value = years, onValueChange = { years = it },
                        valueRange = 1f..50f, steps = 48,
                        modifier = Modifier.fillMaxWidth()
                    )
                    Row(Modifier.fillMaxWidth(), horizontalArrangement = Arrangement.SpaceBetween) {
                        Text("1 yr", style = MaterialTheme.typography.bodySmall, color = MaterialTheme.colorScheme.onSurfaceVariant)
                        Text("50 yr", style = MaterialTheme.typography.bodySmall, color = MaterialTheme.colorScheme.onSurfaceVariant)
                    }
                }
            }
        }

        // ── Results ───────────────────────────────────────────────────────────
        result?.let { res ->
            Card(
                modifier = Modifier.fillMaxWidth(),
                shape = RoundedCornerShape(20.dp),
                colors = CardDefaults.cardColors(containerColor = MaterialTheme.colorScheme.primaryContainer)
            ) {
                Column(modifier = Modifier.padding(16.dp), verticalArrangement = Arrangement.spacedBy(8.dp)) {
                    Text("Results", style = MaterialTheme.typography.titleMedium)

                    ResultRow(
                        "Total Amount (A)", currency.format(res.total),
                        MaterialTheme.colorScheme.primary, large = true
                    )
                    HorizontalDivider(modifier = Modifier.padding(vertical = 2.dp))
                    ResultRow("Principal (P)", currency.format(res.principal), MaterialTheme.colorScheme.onPrimaryContainer)
                    if (res.contributions > 0.01)
                        ResultRow("Contributions", currency.format(res.contributions), MaterialTheme.colorScheme.onPrimaryContainer)
                    ResultRow("Interest Earned (I)", currency.format(res.interest), MaterialTheme.colorScheme.tertiary)

                    val pct = if (res.principal > 0) res.interest / res.principal * 100 else 0.0
                    Text(
                        "Your money grew ${String.format("%.1f", pct)}% on top of initial principal",
                        style = MaterialTheme.typography.bodySmall,
                        color = MaterialTheme.colorScheme.onPrimaryContainer.copy(alpha = 0.7f)
                    )
                }
            }

            // ── Growth Chart ──────────────────────────────────────────────────
            Card(modifier = Modifier.fillMaxWidth(), shape = RoundedCornerShape(20.dp)) {
                Column(modifier = Modifier.padding(16.dp)) {
                    Row(verticalAlignment = Alignment.CenterVertically) {
                        Icon(Icons.Default.TrendingUp, contentDescription = null, tint = MaterialTheme.colorScheme.primary)
                        Spacer(Modifier.width(8.dp))
                        Text("Growth Over Time", style = MaterialTheme.typography.titleMedium)
                    }
                    Spacer(Modifier.height(8.dp))
                    Row(horizontalArrangement = Arrangement.spacedBy(16.dp)) {
                        LegendDot(MaterialTheme.colorScheme.primary, "Total Value")
                        LegendDot(MaterialTheme.colorScheme.secondary, "Amount Invested")
                    }
                    Spacer(Modifier.height(4.dp))

                    val primaryArgb = MaterialTheme.colorScheme.primary.toArgb()
                    val secondaryArgb = MaterialTheme.colorScheme.secondary.toArgb()
                    val onSurfaceArgb = MaterialTheme.colorScheme.onSurface.toArgb()

                    AndroidView(
                        factory = { ctx ->
                            LineChart(ctx).apply {
                                description.isEnabled = false
                                legend.isEnabled = false
                                setTouchEnabled(true)
                                setDrawGridBackground(false)
                                setBackgroundColor(android.graphics.Color.TRANSPARENT)
                                xAxis.apply {
                                    position = XAxis.XAxisPosition.BOTTOM
                                    granularity = 1f
                                    textColor = onSurfaceArgb
                                    setDrawGridLines(false)
                                    valueFormatter = object : ValueFormatter() {
                                        override fun getFormattedValue(v: Float) = "Yr${v.toInt()}"
                                    }
                                }
                                axisLeft.apply {
                                    textColor = onSurfaceArgb
                                    setDrawGridLines(true)
                                    valueFormatter = object : ValueFormatter() {
                                        override fun getFormattedValue(v: Float) = when {
                                            v >= 1_000_000 -> "\$${String.format("%.1f", v / 1_000_000)}M"
                                            v >= 1_000 -> "\$${String.format("%.0f", v / 1_000)}K"
                                            else -> "\$${v.toInt()}"
                                        }
                                    }
                                }
                                axisRight.isEnabled = false
                            }
                        },
                        update = { chart ->
                            val totalEntries = res.yearly.map { (yr, _, tot) -> Entry(yr.toFloat(), tot.toFloat()) }
                            val investedEntries = res.yearly.map { (yr, inv, _) -> Entry(yr.toFloat(), inv.toFloat()) }

                            fun makeSet(entries: List<Entry>, color: Int, filled: Boolean) =
                                LineDataSet(entries, "").apply {
                                    this.color = color
                                    setDrawCircles(false)
                                    lineWidth = 2.5f
                                    mode = LineDataSet.Mode.CUBIC_BEZIER
                                    setDrawValues(false)
                                    if (filled) {
                                        setDrawFilled(true)
                                        fillColor = color
                                        fillAlpha = 50
                                    }
                                }

                            chart.data = LineData(
                                makeSet(totalEntries, primaryArgb, true),
                                makeSet(investedEntries, secondaryArgb, true)
                            )
                            chart.animateX(600)
                            chart.invalidate()
                        },
                        modifier = Modifier.fillMaxWidth().height(220.dp)
                    )
                }
            }

            // ── What If ───────────────────────────────────────────────────────
            Card(
                modifier = Modifier.fillMaxWidth(),
                shape = RoundedCornerShape(20.dp),
                colors = CardDefaults.cardColors(containerColor = MaterialTheme.colorScheme.secondaryContainer)
            ) {
                Column(modifier = Modifier.padding(16.dp), verticalArrangement = Arrangement.spacedBy(8.dp)) {
                    Text("💡 What If?", style = MaterialTheme.typography.titleMedium)
                    Text(
                        "What if you started ${whatIfExtraYears.roundToInt()} year(s) earlier?",
                        style = MaterialTheme.typography.bodyMedium,
                        color = MaterialTheme.colorScheme.onSecondaryContainer
                    )
                    Slider(
                        value = whatIfExtraYears, onValueChange = { whatIfExtraYears = it },
                        valueRange = 0f..20f, steps = 19,
                        modifier = Modifier.fillMaxWidth()
                    )
                    whatIfResult?.also { wif ->
                        val extra = wif.total - res.total
                        Text(
                            "You'd have ${currency.format(wif.total)}, that's ${currency.format(extra)} more!",
                            style = MaterialTheme.typography.bodyMedium,
                            fontWeight = FontWeight.Bold,
                            color = MaterialTheme.colorScheme.tertiary
                        )
                        Text(
                            "Extra interest: ${currency.format(wif.interest - res.interest)}",
                            style = MaterialTheme.typography.bodySmall,
                            color = MaterialTheme.colorScheme.onSecondaryContainer
                        )
                    } ?: Text(
                        "Drag the slider to see the power of starting early",
                        style = MaterialTheme.typography.bodySmall,
                        color = MaterialTheme.colorScheme.onSecondaryContainer.copy(alpha = 0.7f)
                    )
                }
            }

            // ── Frequency Comparison ──────────────────────────────────────────
            Card(modifier = Modifier.fillMaxWidth(), shape = RoundedCornerShape(20.dp)) {
                Column(modifier = Modifier.padding(16.dp), verticalArrangement = Arrangement.spacedBy(4.dp)) {
                    Text("📊 Compounding Frequency Impact", style = MaterialTheme.typography.titleMedium)
                    Text(
                        "Same inputs, different compounding — notice the difference",
                        style = MaterialTheme.typography.bodySmall,
                        color = MaterialTheme.colorScheme.onSurfaceVariant
                    )
                    Spacer(Modifier.height(4.dp))
                    val p = principal.toDoubleOrNull() ?: 0.0
                    val r = rate.toDoubleOrNull() ?: 0.0
                    val pmt = monthly.toDoubleOrNull() ?: 0.0
                    CompoundFrequency.entries.forEach { f ->
                        val r2 = compound(p, r, f, years.toDouble(), pmt)
                        FreqRow(f.label, currency.format(r2.total), isSelected = f == freq)
                    }
                }
            }
        }

        Spacer(Modifier.height(24.dp))
    }
}

// ── Reusable composables ──────────────────────────────────────────────────────

@Composable
fun MoneyField(value: String, label: String, onValueChange: (String) -> Unit, modifier: Modifier = Modifier, prefix: String? = null, suffix: String? = null) {
    OutlinedTextField(
        value = value,
        onValueChange = { new ->
            val digits = new.filter { ch -> ch.isDigit() || ch == '.' }
            val sanitised = if (digits.count { ch -> ch == '.' } <= 1) digits else value
            onValueChange(sanitised)
        },
        label = { Text(label) },
        prefix = prefix?.let { { Text(it) } },
        suffix = suffix?.let { { Text(it) } },
        keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Decimal),
        singleLine = true,
        modifier = modifier
    )
}

@Composable
fun ResultRow(label: String, value: String, color: Color, large: Boolean = false) {
    Row(Modifier.fillMaxWidth(), horizontalArrangement = Arrangement.SpaceBetween, verticalAlignment = Alignment.CenterVertically) {
        Text(
            label,
            style = if (large) MaterialTheme.typography.bodyLarge else MaterialTheme.typography.bodyMedium,
            color = MaterialTheme.colorScheme.onPrimaryContainer
        )
        Text(
            value,
            style = if (large) MaterialTheme.typography.titleLarge else MaterialTheme.typography.bodyLarge,
            fontWeight = FontWeight.Bold,
            color = color
        )
    }
}

@Composable
fun FreqRow(label: String, value: String, isSelected: Boolean) {
    val bg = if (isSelected) MaterialTheme.colorScheme.primary.copy(alpha = 0.1f) else Color.Transparent
    Row(
        modifier = Modifier.fillMaxWidth().background(bg, RoundedCornerShape(8.dp)).padding(horizontal = 8.dp, vertical = 4.dp),
        horizontalArrangement = Arrangement.SpaceBetween
    ) {
        Text(
            label,
            style = MaterialTheme.typography.bodySmall,
            fontWeight = if (isSelected) FontWeight.Bold else FontWeight.Normal,
            color = if (isSelected) MaterialTheme.colorScheme.primary else MaterialTheme.colorScheme.onSurface
        )
        Text(
            value,
            style = MaterialTheme.typography.bodySmall,
            fontWeight = if (isSelected) FontWeight.Bold else FontWeight.Normal,
            color = if (isSelected) MaterialTheme.colorScheme.primary else MaterialTheme.colorScheme.onSurface
        )
    }
}

@Composable
fun LegendDot(color: Color, label: String) {
    Row(verticalAlignment = Alignment.CenterVertically) {
        Box(Modifier.size(10.dp).background(color, RoundedCornerShape(2.dp)))
        Spacer(Modifier.width(4.dp))
        Text(label, style = MaterialTheme.typography.bodySmall)
    }
}
