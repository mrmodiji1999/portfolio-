# portfolio-
https://mrmodiji1999.github.io/portfolio-/
https://medium.com/nerd-for-tech/pagination-in-android-with-paging-3-retrofit-and-kotlin-flow-2c2454ff776e

https://github.com/saumya1singh/30DaysOfAppDev-Course.git



package com.example.a26api

import android.content.Context
import android.os.Bundle
import android.util.Log
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.a26api.sampledata.MyData
import com.example.a26api.sampledata.Product
import com.example.a26api.sampledata.Products
import retrofit2.Call
import retrofit2.Callback
import retrofit2.Response
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

class MainActivity : AppCompatActivity() {

    private lateinit var recyclerView: RecyclerView
    private lateinit var myAdapter: MyAdapter
    private var currentPage = 1
    private var isLoading = false

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        recyclerView = findViewById(R.id.recyclerView)

        // Initialize RecyclerView
        myAdapter = MyAdapter(this, mutableListOf())
        recyclerView.adapter = myAdapter
        recyclerView.layoutManager = LinearLayoutManager(this)

        // Load initial data
        loadData()

        // Add scroll listener for pagination
        recyclerView.addOnScrollListener(object : RecyclerView.OnScrollListener() {
            override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
                super.onScrolled(recyclerView, dx, dy)
                val layoutManager = recyclerView.layoutManager as LinearLayoutManager
                val visibleItemCount = layoutManager.childCount
                val totalItemCount = layoutManager.itemCount
                val firstVisibleItemPosition = layoutManager.findFirstVisibleItemPosition()

                if (!isLoading && visibleItemCount + firstVisibleItemPosition >= totalItemCount && firstVisibleItemPosition >= 0) {
                    currentPage++
                    loadData()
                }
            }
        })
    }

    private fun loadData() {
        val retrofitBuilder = Retrofit.Builder()
            .baseUrl("https://test.hijab-deutschland.com/api/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiInterface::class.java)

        val retrofitData = retrofitBuilder.getProductData(
            currentPage,
            1, // Assuming other parameters remain constant
            "", // Pass customer ID if available
            32,
            392,
            1,
            "BKBGZ6IEC4FEME5ZTBEZS5SSQC4VSX9X",
            "json"
        )

        retrofitData.enqueue(object : Callback<MyData?> {

            override fun onResponse(call: Call<MyData?>, response: Response<MyData?>) {
                val responseBody = response.body()
                responseBody?.products?.allproducts?.product?.let { productList ->
                    myAdapter.addProducts(productList)
                }
                isLoading = false
            }

            override fun onFailure(call: Call<MyData?>, t: Throwable) {
                // Handle failure
                Log.d("Main Activity ", "onFailure: " + t.message)
                isLoading = false
            }
        })
    }

    private class MyAdapter(private val context: Context, private val productList: MutableList<Product>) : RecyclerView.Adapter<MyAdapter.ViewHolder>() {

        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val view = LayoutInflater.from(parent.context).inflate(R.layout.activity_main, parent, false)
            return ViewHolder(view)
        }

        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            val product = productList[position]
            holder.bind(product)
        }

        override fun getItemCount(): Int {
            return productList.size
        }

        fun addProducts(newProducts: List<Product>) {
            val startPos = productList.size
            productList.addAll(newProducts)
            notifyItemRangeInserted(startPos, newProducts.size)
        }

        inner class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
            private val productNameTextView: TextView = itemView.findViewById(R.id.textView)

            fun bind(product: Product) {
                productNameTextView.text = product.name

            }
        }
    }
}
