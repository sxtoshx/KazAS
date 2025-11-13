## KazAS
```
package com.example.slotmachine

import android.os.Bundle
import android.view.View
import android.view.animation.AccelerateDecelerateInterpolator
import android.view.animation.DecelerateInterpolator
import android.view.animation.LinearInterpolator
import android.widget.Button
import android.widget.ImageView
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import kotlin.random.Random

class MainActivity : AppCompatActivity() {

    private lateinit var ivReel1: ImageView
    private lateinit var ivReel2: ImageView
    private lateinit var ivReel3: ImageView
    private lateinit var btnSpin: Button
    private lateinit var btnRestart: Button
    private lateinit var tvBalance: TextView
    private lateinit var tvResult: TextView

    private var balance = 100

    private val symbols = listOf(
        R.drawable.symbol_1,
        R.drawable.symbol_2,
        R.drawable.symbol_3,
        R.drawable.symbol_4,
        R.drawable.symbol_5,
        R.drawable.symbol_6
    )

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        ivReel1 = findViewById(R.id.ivReel1)
        ivReel2 = findViewById(R.id.ivReel2)
        ivReel3 = findViewById(R.id.ivReel3)
        btnSpin = findViewById(R.id.btnSpin)
        btnRestart = findViewById(R.id.btnRestart)
        tvBalance = findViewById(R.id.tvBalance)
        tvResult = findViewById(R.id.tvResult)

        updateBalanceText()

        btnSpin.setOnClickListener { spin() }
        btnRestart.setOnClickListener { restartGame() }
    }

    private fun updateBalanceText() {
        tvBalance.text = "Баланс: $balance"
    }

    private fun restartGame() {
        balance = 100
        updateBalanceText()
        tvResult.text = ""
        btnSpin.visibility = View.VISIBLE
        btnRestart.visibility = View.GONE
    }

    private fun spin() {
        if (balance <= 0) {
            tvResult.text = "У тебя ноль монет — игра окончена."
            btnSpin.visibility = View.GONE
            btnRestart.visibility = View.VISIBLE
            return
        }

        val bet = 15
        balance -= bet
        updateBalanceText()
        tvResult.text = "Крутим барабаны..."
        btnSpin.isEnabled = false

        val i1 = Random.nextInt(symbols.size)
        val i2 = Random.nextInt(symbols.size)
        val i3 = Random.nextInt(symbols.size)

        
        animateReel(ivReel1, 1700L, i1) {}
        animateReel(ivReel2, 2000L, i2) {}
        animateReel(ivReel3, 2300L, i3) {
            showResult(i1, i2, i3, bet)
            btnSpin.isEnabled = true
            if (balance <= 0) {
                btnSpin.visibility = View.GONE
                btnRestart.visibility = View.VISIBLE
            }
        }
    }

    
    private fun animateReel(view: ImageView, duration: Long, finalIndex: Int, onEnd: () -> Unit) {
        val startTime = System.currentTimeMillis()
        val baseDelay = 45L
        val totalSymbols = symbols.size

        val runnable = object : Runnable {
            override fun run() {
                val elapsed = System.currentTimeMillis() - startTime
                val progress = (elapsed.toFloat() / duration).coerceIn(0f, 1f)
                val easeOut = 1f - (1f - progress)
                val currentDelay = (baseDelay + easeOut * 110).toLong()

               
                view.animate()
                    .rotationX(90f)
                    .setDuration(70)
                    .setInterpolator(LinearInterpolator())
                    .withEndAction {
                        val randomIndex = Random.nextInt(totalSymbols)
                        view.setImageResource(symbols[randomIndex])

                        
                        val blurFactor = (1f - progress) * 0.6f
                        view.alpha = 1f - blurFactor * 0.4f
                        view.scaleX = 1f + blurFactor * 0.3f
                        view.scaleY = 1f + blurFactor * 0.3f

                        view.rotationX = -90f
                        view.animate()
                            .rotationX(0f)
                            .setDuration(70)
                            .setInterpolator(DecelerateInterpolator())
                            .start()
                    }
                    .start()

                if (elapsed < duration) {
                    view.postDelayed(this, currentDelay)
                } else {
                    
                    view.animate()
                        .rotationX(90f)
                        .setDuration(120)
                        .setInterpolator(AccelerateDecelerateInterpolator())
                        .withEndAction {
                            view.setImageResource(symbols[finalIndex])
                            view.alpha = 1f
                            view.scaleX = 1f
                            view.scaleY = 1f
                            view.rotationX = -90f
                            view.animate()
                                .rotationX(0f)
                                .setDuration(120)
                                .setInterpolator(DecelerateInterpolator())
                                .withEndAction {
                                    view.alpha = 1f
                                    view.scaleX = 1f
                                    view.scaleY = 1f
                                    onEnd()
                                }
                                .start()
                        }
                        .start()
                }
            }
        }

        view.post(runnable)
    }

    private fun showResult(i1: Int, i2: Int, i3: Int, bet: Int) {
        val resultText: String = when {
            i1 == i2 && i2 == i3 -> {
                val win = bet * 3
                balance += win
                "ДЖЕКПОТ! Ты выиграл $win."
            }
            i1 == i2 || i2 == i3 || i1 == i3 -> {
                val win = bet
                balance += win
                "Два совпали! +$win."
            }
            else -> "Нет совпадений."
        }

        updateBalanceText()
        tvResult.text = resultText
    }
}
```
