require 'httparty'
#require 'json'
#rumus :
# => deg = (rad * 180)/ phi
# => rad = (deg * phi)/ 180
# => 1 mile = 1609.344 m
# => R = 6371 Km (earth mean radius)


class KoprolBridge < ActiveRecord::Base
  include HTTParty
  
  base_uri 'www.koprol.com'
  KOPROL_URI = "http://www.koprol.com"
  API_VERSION = '/api/v2'
  TO_RAD = Math::PI / 180
  LAYER_NAME = "hackandroll"
  Rkm = 6371
  Rmeter = Rkm * 1000
  
  #TYPE
  TOP_NEARBY = "1"
  NEARBY_STREAMS = "2"
  
  def self.get_top_nearby(lat, lng, max_object=10,page=1)
    result = get("#{API_VERSION}/places/top_nearby.json?lat=#{lat}&lng=#{lng}&page=#{page}&per_page=#{max_object}")
    places = result["places"].to_a
    
    response = {}
    response["layer"] = LAYER_NAME
    response["hotspots"] = Array.new
    if places.empty?
      response["hotspots"] = Array.new
      response["errorCode"] = 20
      response["errorString"] = "No POI found. Duh no interesting point in your area."
    else
      response["errorCode"] = 0
  		response["errorString"] = "ok"
      for place in places do
        place = place["place"]
        poi = {}
        poi["actions"] = Array.new
        
        unless place["phone"].blank?
          poi["actions"] << {"label" => "Call", 
            "uri" => "tel:#{phone_clear_symbol(place["phone"])}",
            "activityType" => 4,
            "contentType" => "application/vnd.layar.internal",
            "closeBiw" => true
            }
        end
        
        unless place["web"].blank?
          poi["actions"] << {"label" => "Open Web",
            "uri" => "#{self.generate_web(place["web"])}",
            "activityType" => 20,
            "contentType" => "text/html",
            "closeBiw" => true
          }
        end
        
        poi["id"] = place["id"]
        poi["title"] = place["name"]
        poi["lat"] = change_to_int_loc(place["lat"])
        poi["lon"] = change_to_int_loc(place["lng"])
      
        poi["type"] = change_to_int(TOP_NEARBY)
        poi["dimension"] = change_to_int("1")
        m_distance = km_to_m(place["distance"])
        poi["distance"] = m_distance.to_f
        poi["imageURL"] = "http://l.yimg.com/on/images/thumbs_up.png"
        poi["line2"] = place["address"] 
        poi["line3"] = "likes: #{place["likes"]}"
        poi["line4"] = "dislikes: #{place["dislikes"]}"
        poi["attribution"] = place["place_type"]
        response["hotspots"] << poi
      end
    end
    return response
  end
  
  def self.get_nearby_streams(lat, lng, max_object=10, max_id=999999, page=1)
    result = get("#{API_VERSION}/places/nearby_streams.json?lat=#{lat}&lng=#{lng}&max_id=#{max_id}&page=#{page}&per_page=#{max_object}")
    streams = result["streams"].to_a
    response = {}
    response["layer"] = LAYER_NAME
    response["hotspots"] = Array.new
    if streams.empty?
      response["hotspots"] = Array.new
      response["errorCode"] = 20
      response["errorString"] = "No POI found. Duh no interesting point in your area."
    else
      response["errorCode"] = 0
  		response["errorString"] = "ok"
      count = 0
      for stream in streams do
        stream = stream["stream"]
        place = stream["place"]
        user = stream["user"]
        poi = {}
        poi["actions"] = Array.new
        poi["actions"] << {"label" => "View Stream", 
                          "uri" => "#{KOPROL_URI}#{stream["permalink"]}",
                          "activityType" => 27,
                          "contentType" => "text/html",
                          "closeBiw" => true
                          }
        poi["id"] = stream["id"]
        poi["title"] = user["username"]
        poi["lat"] = change_to_int_loc(place["lat"])
        poi["lon"] = change_to_int_loc(place["lng"])
        
        poi["type"] = change_to_int(NEARBY_STREAMS)
        poi["dimension"] = change_to_int("1")
        m_distance = km_to_m(get_distance(lat.to_f, lng.to_f, place["lat"], place["lng"]))
        poi["distance"] = m_distance.to_f
        poi["imageURL"] = user["full_avatar_url"]
        poi["line2"] = stream["body"] 
        poi["line3"] = stream["date"]
        poi["line4"] = "#{place["name"]} - #{place["city"]}"
        poi["attribution"] = "#{KOPROL_URI}#{stream["permalink"]}"
        response["hotspots"] << poi
        count+=1
      end
    end
    return response
  end
  
  private
#   compute distance using haversine formula
#   Haversine formula:
#   a = sin²(Δlat/2) + cos(lat1).cos(lat2).sin²(Δlong/2)
#   c = 2.atan2(√a, √(1−a))
#   d = R.c
#  	where R is earth’s radius (mean radius = 6,371km);
#   note that angles need to be in radians to pass to trig functions!
    def self.get_distance(lat1, lon1, lat2, lon2)
      dlon = dlon_rad(lon2, lon1)
      dlat = dlat_rad(lat2, lat1)
      lat1 = get_rad(lat1)
      lat2 = get_rad(lat2)
        
      a = (Math.sin(dlat/2))**2 + (Math.cos(lat1) * Math.cos(lat2) * (Math.sin(dlon/2))**2) 
      c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a))
      d = Rkm * c
      return d
    end
    
    def self.change_to_int(data)
      if(data.strip.length > 0)
        return data.to_i
      else
        return nil
      end
    end
    
    # Convert a decimal GPS latitude or longitude value to an integer by multiplying by 1000000.
    def self.change_to_int_loc(location)
      location * 1000000
    end
    
    def self.change_to_float(data)
      if(data.length > 0)
        return data.to_f
      else
        return nil
      end
    end
    
    def self.dlon_rad(lon2, lon1)
      return get_rad(lon2 - lon1)
    end
    
    def self.dlat_rad(lat2, lat1)
      return get_rad(lat2 - lat1)
    end
    
    def self.get_rad(deg)
      return deg * TO_RAD
    end
    
    def self.km_to_m(kilometer)
      return kilometer * 1000
    end
    
    def self.mile_to_m(mile)
      return mile * 1609.344
    end
   
    def self.phone_clear_symbol(phone)
      phone.gsub(/[- ]/,'')
    end
    
    def self.generate_web(web)
      if !web.include? "http://"
        return web.insert(0,"http://")
      else
        return web
      end
    end
end
