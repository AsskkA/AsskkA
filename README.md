function add_product_schema_json_ld() {
    if (is_product()) {
        global $product;

        // Основна інформація про продукт
        $productName = $product->get_name();
        $description = $product->get_description() ?: 'Опис продукту відсутній.';
        $brandName = $product->get_attribute('pa_brend') ?: 'Бренд не вказано'; // Динамічне отримання назви бренду
        $minDeliveryDays = 1;
        $maxDeliveryDays = 3;
        $isOnSale = $product->is_on_sale();
        $priceValidUntil = $isOnSale && $product->get_date_on_sale_to() ? $product->get_date_on_sale_to()->date('Y-m-d') : '';

        // Створення основної структури даних
        $schema_data = [
            "@context" => "http://schema.org/",
            "@type" => "Product",
            "name" => $productName,
            "description" => $description,
            "brand" => [
                "@type" => "Brand",
                "name" => $brandName
            ],
            "image" => wp_get_attachment_image_url($product->get_image_id(), 'full'),
            "address" => [
                "@type" => "PostalAddress",
                "streetAddress" => "вул. Полтавський шлях, будинок 84/40",
                "addressLocality" => "Харків",
                "addressRegion" => "Харківська область",
                "postalCode" => "61093",
                "addressCountry" => "UA"
            ]
        ];

        // Тільки додаємо AggregateRating якщо є відгуки
        if ($product->get_review_count() > 0) {
            $schema_data['aggregateRating'] = [
                "@type" => "AggregateRating",
                "ratingValue" => $product->get_average_rating(),
                "reviewCount" => $product->get_review_count()
            ];
        }

        // Деталі доставки і повернення
        $freeShippingThreshold = 600;  // Сума для безкоштовної доставки може змінюватися
        $shippingCost = ($product->get_price() > $freeShippingThreshold) ? 0 : 50; // Припустима логіка для вартості доставки

        $schema_data['offers'] = [
            "@type" => "Offer",
            "price" => $product->get_price(),
            "priceCurrency" => get_woocommerce_currency(),
            "itemCondition" => "http://schema.org/NewCondition",
            "availability" => "http://schema.org/InStock",
            "priceValidUntil" => $priceValidUntil,
            "shippingDetails" => [
                "@type" => "OfferShippingDetails",
                "shippingRate" => [
                    "@type" => "MonetaryAmount",
                    "value" => $shippingCost,
                    "currency" => "UAH"
                ],
                "shippingDestination" => [
                    "@type" => "DefinedRegion",
                    "name" => "UA",
					"addressCountry" => [
                       "@type" => "Country",
                       "name" => "UA"
                  ]
                ],
                "handlingTime" => [
                    "@type" => "QuantitativeValue",
                    "minValue" => 1, // Мінімальна кількість днів
                    "maxValue" => 3, // Максимальна кількість днів
                    "unitCode" => "DAY"
                ],
                "deliveryTime" => [
                    "@type" => "ShippingDeliveryTime",
                    "businessDays" => [
                        "@type" => "OpeningHoursSpecification",
                        "dayOfWeek" => ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"],
                        "opens" => "09:00",
                        "closes" => "18:00"
                    ],
                    "transitTime" => [
                        "@type" => "QuantitativeValue",
                        "minValue" => $minDeliveryDays,
                        "maxValue" => $maxDeliveryDays,
                        "unitCode" => "DAY"
                    ],
					"handlingTime" => [
                        "@type" => "QuantitativeValue",
                        "minValue" => 1, // Мінімальна кількість днів
                        "maxValue" => 3, // Максимальна кількість днів
                        "unitCode" => "DAY" // Одиниця вимірювання - дні
                    ]

                ],
			 ],
            "hasMerchantReturnPolicy" => [
                "@type" => "MerchantReturnPolicy",
                "refundType" => "http://schema.org/FullRefund",
                "returnWithin" => "P14D",
                "returnPolicyCategory" => "http://schema.org/MerchantReturnFiniteReturnWindow",
                "itemCondition" => "http://schema.org/NewCondition",
                "restockingFee" => [
                    "@type" => "MonetaryAmount",
                    "value" => "0",
                    "currency" => "UAH"
                ],
                "returnPolicyCountry" => "UA",
                "applicableCountry" => "UA",
                "returnFees" => "http://schema.org/ReturnShippingFees",
                "returnMethod" => "http://schema.org/ReturnByMail",
                "merchantReturnDays" => 14,
                "merchantReturnLink" => "mailto:no-reply@ecolavka.kh.ua"
            ]
        ];

        // Вивід JSON-LD скрипту
        echo '<script type="application/ld+json">' . json_encode($schema_data, JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT) . '</script>';
    }
}

