class PoiController < ApplicationController
  def index
    redirect_to root_path
  end
  
  def action
    lat = params[:lat]
    lng = params[:lon]
    page = params[:page]
    max_id = params[:max_id]
    type = params[:RADIOLIST]
    max_object = params[:CUSTOM_SLIDER].to_i if params[:CUSTOM_SLIDER]
    result = nil
    if type
      case type
      when KoprolBridge::TOP_NEARBY
        puts "top nearby..."
        result = KoprolBridge.get_top_nearby(lat, lng, max_object, page)
      when KoprolBridge::NEARBY_STREAMS
        puts "nearby streams..."
        result = KoprolBridge.get_nearby_streams(lat, lng, max_object, max_id, page)
      end
    else
      result = {"error" => "please specify type"}
    end
    
    render :json => result.to_json
  end 
end
