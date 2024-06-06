# Task 2
Code for serializers.py.
I prefer to divide one big serialezer to several more smaller to incaplusate business logic into
```python

logger = logging.getLogger(__name__)

class AdCreativeSerializer(ModelSerializer):
    class Meta:
        model = AdCreative
        fields = ['fb_id', 'title', ...]


class AdSerializer(ModelSerializer):
    active_ad_creative = AdCreativeSerializer()

    class Meta:
        model = Ad
        fields = ['fb_id', ...]

    def create(self, validated_data):
        '''
        We get or create an active ad_creative and associate it with ad
        '''
        active_ad_creative_data = validated_data.pop('active_ad_creative')
        active_ad_creative, _ = AdCreative.objects.get_or_create(fb_id=active_ad_creative_data['fb_id'], defaults=active_ad_creative_data)
        ad = Ad.objects.create(active_ad_creative=active_ad_creative, **validated_data)

        logger.info(f"New Ad created: {ad.fb_id}")
        return ad

class AdsetSerializer(ModelSerializer):
    ad_list = AdSerializer(many=True)

    class Meta:
        model = Adset
        fields = ['fb_id', 'ad_campaign', ... ]

    def create(self, validated_data):
        '''
        Creating each ad for an ad set
        '''
        ad_list_data = validated_data.pop('ad_list')
        adset = Adset.objects.create(**validated_data)
        for ad_data in ad_list_data:
            Ad.objects.create(adset=adset, **ad_data)

        logger.info(f"New Adset created: some info and date")
        return adset

class AdCampaignSerializer(ModelSerializesr):
    adset_list = AdsetSerializer(many=True)

    class Meta:
        model = AdCampaign
        fields = ['fb_id', 'name', 'status'... ]

    def create(self, validated_data):
        '''
        Creating each ad set for an ad campaign
        '''
        adset_list_data = validated_data.pop('adset_list')
        ad_campaign = AdCampaign.objects.create(**validated_data)
        for adset_data in adset_list_data:
            Adset.objects.create(ad_campaign=ad_campaign, **adset_data)

        logger.info(f"New AdCampaign created: some infoa and date")
        return ad_campaign
```

And here, we need some function to varify json body, that will be utilizing in views.py:
```python
    def process_json_data(json_data):
        ad_campaigns_data = json_data.get('ad_campaigns', [])
        ad_creatives_data = json_data.get('ad_creatives', [])

        for ad_campaign_data in ad_campaigns_data:
            ad_campaign_serializer = AdCampaignSerializer(data=ad_campaign_data)
            if ad_campaign_serializer.is_valid():
                ad_campaign_serializer.save()
            else:
                logger.info(f"Invalid data: Some data values, date of error etc")
                raise ValidationError(ad_campaign_serializer.errors, code=400)

        for ad_creative_data in ad_creatives_data:
            ad_creative_serializer = AdCreativeSerializer(data=ad_creative_data)
            if ad_creative_serializer.is_valid():
                ad_creative_serializer.save()
            else:
                logger.info(f"Invalid data: Some data values, date of error etc")
                raise ValidationError(ad_creative_serializer.errors, code=400)
```